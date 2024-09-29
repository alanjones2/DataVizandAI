---
layout: post
image: https://cdn-images-1.medium.com/max/1200/1*BhucSzDp94gE2iZLAgrW4g.png
title: From Data to Visualization with the OpenAI Assistants API and GPT-4o
excerpt: We explore the Code Completion tool from OpenAI’s Assistants API to create visualizations directly from data
---


## We explore the Code Completion tool from OpenAI’s Assistants API to create visualizations directly from data

![](https://cdn-images-1.medium.com/max/1200/1*BhucSzDp94gE2iZLAgrW4g.png)

*Programming tools — Image constructed from photos by [Quaritsch Photography](https://unsplash.com/@quaritsch?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) and [Anton Savinov](https://unsplash.com/@tonchik?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

As GPT-4’s capabilities continue to expand, OpenAI’s tools built on its technology are evolving into increasingly powerful assets for developers.

In this article, we are going to explore the chart-making capabilities of the latest iteration. We’ll equip an Assistant with a data file and specific instructions to see how it brings our data visualization ideas into being.

We will use the built-in tools in the Assistant API to achieve this.

Currently, in the OpenAI Python package (v1.30.0, at the time of writing) the Assistants API includes the tools, File Search, Code Completion and Function Calling.

Function Calling lets the developer define functions that the AI can intelligently choose in order to complete a task; File Search allows the developer to upload a variety of file types which can be stored, RAG style, in a vector database; and Code Completion lets the assistant write and run Python programs in a sandboxed environment to solve programming and math problems.

Code Completion can also work with uploaded files, which can be processed to create data files and images of charts. And that is the functionality that we will be using here.

The code that we will explore below will load a data file (in CSV form), and by using appropriate prompts, instruct our assistant to create a graph from the data. We will then download the graph and display it.

Using plain English prompts, we can easily generate charts such as the ones below from raw CSV data.

![](https://cdn-images-1.medium.com/max/1200/1*EWQK128vsaUAwZHEMr3xfw.png)

### OpenAI Assistants

I have explained OpenAI’s Assistants API, and how to get started with them previously ([*Use OpenAI’s Powerful New Assistants API for Data Analysis*](https://medium.com/towards-data-science/how-to-use-the-powerful-new-assistants-api-for-data-analysis-c9ea1cab0b53)). While the new versions of the API have rendered this article a little out of date, the description of Assistants and how they work is still broadly correct, as is how to get up and running with an OpenAI account.

So, for a more detailed look, you can refer to that article and here I will limit myself to a brief introduction of the basics before getting into the nitty-gritty.

### OpenAI

Firstly, of course, you will need an OpenAI account, and you need to be aware that you will be charged for using it. However, the fees are not high: running the code we will describe here once, should cost only a few cents. Other charges apply, such as file storage, and while they are not likely to be relevant in this context, you should check up on the latest fees.

Having said that, you should regularly use the OpenAI dashboard to check your usage so as to make sure you are not racking up a large bill.

![](https://cdn-images-1.medium.com/max/800/1*MxXiUczWlKIq9FPqc1nJsw.png)

_The OpenAI dashboard menu_

You should also use the dashboard to check the storage that you are using. All the output from an assistant is stored along with any files that you have uploaded. You can delete them manually from the dashboard, and you probably want to do this because while you may not be charged, over time a lot of unnecessary files can accumulate in your workspace.

### Assistants, Threads, and Runs

These are the three basic objects in the Assistants API.

- `Assistants:` As you might imagine, these are the fundamental part of the setup. When we create an assistant, we specify various attributes such as: a model (e.g. `gpt-4o`); instructions that inform the model about the type of behaviour we expect from it; tools such as the *code interpreter* and *file search*; and files that we want the model to use.
- `Threads:` These store the state of a conversation and contain the `messages` that are generated both by the user and the assistant. A thread is associated with an assistant when a `run` is started (see below).
- `Runs:` These control the execution of an `assistant` with a `thread`. The `run` takes the information in the `thread` and the `assistant` and manages the interaction with the LLM. `Runs` go through a number of steps before completion. When the `run` is complete, we can interrogate the `thread` to see what response the assistant has come up with.

In addition to these basic objects, in a thread, we need `messages` that will contain the instructions for the model and its responses. Also, we shall use `files` which are separate objects that assistants use and which hold the details of files that are uploaded.

### Coding the Assistant

We will need to go through a number of steps to create and run our assistant. The sequence of events is listed below and gives an overview of how each of the components is used.

The steps are:

- Create an OpenAI client with our API key.
- Upload a local file and retrieve the file object for later use.
- Create an assistant with instructions for the model and the ID of the uploaded file.
- Create a thread that also includes the file ID and instructions for the model.
- Run the assistant and thread.
- Display the messages generated by both the user and the AI that are now in the thread (this should show the process the model went through to produce a result, and if anything went wrong we can, hopefully, see what it was).
- Retrieve the generated image and display it.

We will code each of the bullet points above in Python and describe exactly what is happening. I’ve written the code in the form of a Jupyter notebook, so if you want to follow along, copy each section of code into a new notebook cell, and you’ll have a copy of my notebook.

The first step is to create a client.

#### Create a client

The client gives us access to the OpenAI API. We need to provide it with an API key and in the code below, I have included an input statement so that you need to enter the key manually.


``` python
from openai import OpenAI  

key = input("API key")  
client = OpenAI(api_key=key)
```

You could also hard code the key (but make sure your code doesn’t go public).

```python
from openai import OpenAI  

client = OpenAI(api_key="Your key here")
```

Or, if your key is stored as an environment variable then the client will find it automatically and all you need to code is…

```python
from openai import OpenAI  

client = OpenAI()
```

One of these is your first Jupyter Notebook cell.

#### Upload a file

First, you need a file! I’m using a CSV file that is derived from data on the [Our World in Data (OWID) website.](https://ourworldindata.org/co2-and-greenhouse-gas-emissions?insight=human-greenhouse-gas-emissions-have-increased-global-average-temperatures#key-insights) OWID is a great source of information and data, and they helpfully allow all of their content to be used freely under the [Creative Commons BY license](https://creativecommons.org/licenses/by/4.0/).

The file records worldwide CO2 emissions from 1850 to 2021 (the original data contained data for many other entries, but I have only included World data, here). You can see what it looks like in the screenshot below.

![](https://cdn-images-1.medium.com/max/1200/1*0WawRuVQMFfpdXqlDakt3Q.png)

I’ve called the file *world_df.csv* and, I also want to set a variable with the name that I'm going to give to the assistant. So, my second notebook cell contains two variables for these values.

```python
filename = "world_df.csv"  
assistant_name = "data-analyst-v0.1"
```

If I want to use the code to read different files or create a new assistant, I can change the values in this cell.

The next cell uploads the file. The main work of uploading a file is done by the `client.files.create` method. In the code below, this method takes two parameters, the file itself and the string ‘assistants’ which tells the client what the purpose of the file is.

The code does a little more than just upload a file. I’m assuming that the code will be run more than once (perhaps with different instructions) and I don’t want to duplicate the files. So, if the file is new, the code uploads it, but if it had been uploaded earlier, the code retrieves that existing file.

```python
# Check if file is already uploaded  
filelist = client.files.list(purpose="assistants")  
filenames =  [x.filename for x in filelist.data]  

# Upload a file with an "assistants" purpose or use existing one  
if not filename in filenames:  
  file = client.files.create(  
    file=open(filename, "rb"),  
    purpose='assistants'  
  )  
else:  
  for f in filelist:  
    if f.filename == filename:  
      file = client.files.retrieve(f.id)  
      break
```

We can check if the file exists by downloading a list of already uploaded files and searching it to see if our file is there. The method `client.files.list()` retrieves the list from the client and we pass the parameter`purpose='assistants'` to show the type of file we are interested in.

We can then scan the list to find the file name of interest. If it is not there, we upload, otherwise we fetch the file object from the client. In either case, `file`is set to the file object.

In an app, this code could be usefully placed in a function that returned the file object.

So, we now have the file uploaded and a record of the file object. Next, we need to create an assistant that will use that file.

#### Create an assistant

As with the uploaded file, we check to see if it already exists. Again, if we have run this code before, then the assistant will have already been created, and we don’t want to duplicate it, and so we just retrieve the existing assistant object. Otherwise, we create a new one.

The code for this functionality is pretty much the same as we used for the file upload.

Creating an assistant, is done with a call to `client.beta.assistants.create().`

We set the parameters for the name of the assistant, some basic instructions (these will be a System Prompt), the model that we want to use (in the case GPT-4o), the tools that we require (code interpreter) and the resources. In this last parameter, we can see that we reference the file object for the uploaded file and indicate that the code interpreter will be using the file.

```python
# Check if assistant already exists  
assistant_list = client.beta.assistants.list()  
assistant_names =  [x.name for x in assistant_list.data]  

if not assistant_name in assistant_names:  
  # Create an assistant using the file ID  
  assistant = client.beta.assistants.create(  
    name = "data-analyst-v0.1",  
    instructions="You are a data analyst",  
    model="gpt-4o",  
    tools=[{"type": "code_interpreter"}],  
    tool_resources={  
      "code_interpreter": {  
        "file_ids": [file.id]  
      }  
    }  
  )  
else:  
    for a in assistant_list:  
      if a.name == assistant_name:  
        assistant = client.beta.assistants.retrieve(a.id)  
        break
```

Again, in an app, this could be a function that returns the assistant object.

#### Create a thread

To create a thread, we simply call `client.beta.threads.create()` and specify the first message that will be passed to the assistant when it runs the assistant with this thread.

As you can see in the code below, in the message we set the role, the prompt, and add the file ID as an attachment.

```python
thread = client.beta.threads.create(  
  messages=[  
    {  
      "role": "user",  
      "content": "Using the csv file attached, display a graph of 'Year' against 'Annual CO2 emissions",  
      "attachments": [  
        {  
          "file_id": file.id,  
          "tools": [{"type": "code_interpreter"}]  
        }  
      ]  
    }  
  ]  
)
```

The prompt that we are sending to the LLM is:

*“Using the csv file attached, display a graph of ‘Year’ against ‘Annual CO2 emissions”.*

That’s a fairly straightforward requirement that requires the code interpreter to analyse the data file and generate the required code.

Now we are all set to run the assistant with the thread.

#### Create a run

A run takes the assistant and the thread and submits them to the LLM. It runs asynchronously and goes through a number of steps before it finishes.

Consequently, we need to wait for a result, and we can do this in two ways: polling or streaming. Polling repeatedly checks the status of the run until it is completed. Whereas with streaming the various steps are automatically detected and functions can be mapped onto those events with an event hander.

Below is the streaming code from the [OpenAI documentation](https://platform.openai.com/docs/assistants/overview?context=with-streaming) (with a changed message) which suffices for this experiment.

```python
from typing_extensions import override  
from openai import AssistantEventHandler  

# First, we create a EventHandler class to define  
# how we want to handle the events in the response stream.  

class EventHandler(AssistantEventHandler):  
  @override  
  def on_text_created(self, text) -> None:  
    print(f"\nassistant > {text.value}", end="", flush=True)  

  @override  
  def on_text_delta(self, delta, snapshot):  
    print(delta.value, end="", flush=True)  

  def on_tool_call_created(self, tool_call):  
    print(f"\nassistant > {tool_call.type}\n", flush=True)  

  def on_tool_call_delta(self, delta, snapshot):  
    if delta.type == 'code_interpreter':  
      if delta.code_interpreter.input:  
        print(delta.code_interpreter.input, end="", flush=True)  
      if delta.code_interpreter.outputs:  
        print(f"\n\noutput >", flush=True)  
        for output in delta.code_interpreter.outputs:  
          if output.type == "logs":  
            print(f"\n{output.logs}", flush=True)  

# Then, we use the `stream` SDK helper  
# with the `EventHandler` class to create the Run  
# and stream the response.  

with client.beta.threads.runs.stream(  
  thread_id=thread.id,  
  assistant_id=assistant.id,  
  instructions="create a downloadable file for the graph",  
  event_handler=EventHandler(),  
) as stream:  
  stream.until_done()
```

The function which starts the run is `client.bet.threads.run.stream()` and we pass the IDs of the thread and the assistant along with instructions for this particular run and the event handler which is defined above.

We won’t go into detail about the workings of the event handler, suffice it to say that it catches events when text is created or a tool outputs something and prints the result. Its functionality is fine for experimental purposes but for a real app, you would probably want to do something more sophisticated with these outputs.

Note that we specified in the thread that we wanted a graph to be generated and here in the run we additionally ask it to generate a downloadable file.

The output from the run is shown below and mostly comprises the Python code generated by the assistant.

```python
assistant > code_interpreter  

import pandas as pd  

# Load the CSV file  
file_path = '/mnt/data/file-8XwqMOlaH6hoKEEKOYXPYqTh'  
data = pd.read_csv(file_path)  

# Display the first few rows of the dataframe to understand its structure  
data.head()import matplotlib.pyplot as plt  

# Plot 'Year' vs 'Annual CO₂ emissions'  
plt.figure(figsize=(10, 6))  
plt.plot(data['Year'], data['Annual CO₂ emissions'], marker='o', linestyle='-')  
plt.xlabel('Year')  
plt.ylabel('Annual CO₂ emissions')  
plt.title('Year vs Annual CO₂ emissions')  
plt.grid(True)  
plt.tight_layout()  

# Save the plot to a file  
plot_file_path = '/mnt/data/year_vs_annual_co2_emissions.png'  
plt.savefig(plot_file_path)  
plot_file_path  

output >  

assistant > The graph depicting 'Year' against 'Annual CO₂ emissions' has been created. You can download the plot using the link below:  

[Download the graph](sandbox:/mnt/data/year_vs_annual_co2_emissions.png)None
```

*To be clear the code above is NOT to be included in the Jupyter Notebook. It was generated and run by GPT.*

The output shows that the LLM has understood our instructions, has generated the code to create the correct graph, has run that code, and has created an image file that we can download.

#### Retrieve the generated file

All we need to do now is find the file that the assistant generated and download it.

The last code cell in our notebook is shown below.

```python
filelist = client.files.list(purpose="assistants_output")  

image_list = [x for x in filelist.data if "png" in x.filename]  

id = image_list[-1].id # the last in the list is the latest  

image_data = client.files.content(id)  
image_data_bytes = image_data.read()  

with open("./my-image.png", "wb") as file:  
    file.write(image_data_bytes)
```

Again, we assume that this (or similar) code may have been run before, so there might well be more than one image file.

So, we first get a list of all the files that are labelled as “assistants_output”, then we create a list of images (i.e. files with the extension ‘.png’) and then we find the last file in that list — that will be the last one that was generated.

And to display the chart we can create a markdown cell with the following content.

```bash
![](my-image.png)
```

The result can be seen in the image below:

![](https://cdn-images-1.medium.com/max/800/1*NQxjzaIMsTw0AueTEQPXUQ.png)

An example of an AI-generated graph: CO2 emissions over time

### Changing the prompt

To generate a different chart we simply run the code again but with a different prompt. For example:

*“Using the csv file attached, display a graph of ‘Year’ against all other columns”*

The result is the image below.

![](https://cdn-images-1.medium.com/max/800/1*01lkKzIVhodN9hZopnTWVA.png)

Or, if we are only interested in the data for this century:

*“Using the csv file attached, display a graph of ‘Year’ against all other columns. Only chart the figures for the 21st century”*

This elicits the response below.

```python
assistant > code_interpreter  

import pandas as pd  

# Load the CSV file  
file_path = '/mnt/data/file-8XwqMOlaH6hoKEEKOYXPYqTh'  
data = pd.read_csv(file_path)  

# Display the first few rows of the dataframe to understand its structure  
data.head()import matplotlib.pyplot as plt  

# Filter the data to include only the 21st century (from year 2000 onwards)  
data_21st_century = data[data['Year'] >= 2000]  

# Define the columns to be plotted against 'Year'  
columns_to_plot = data.columns.drop(['Entity', 'Code', 'Year'])  

# Plot the data  
plt.figure(figsize=(12, 8))  
for column in columns_to_plot:  
    plt.plot(data_21st_century['Year'], data_21st_century[column], label=column)  

plt.title('Yearly Data in the 21st Century')  
plt.xlabel('Year')  
plt.ylabel('Values')  
plt.legend(title='Metrics', bbox_to_anchor=(1.05, 1), loc='upper left')  
plt.grid(True)  
plt.tight_layout()  

# Save the plot to a file  
plot_file_path = '/mnt/data/my-image3.png'  
plt.savefig(plot_file_path)  

plt.show()  

plot_file_path  

output >  

assistant > The graph has been created and saved successfully. You can download the file using the link below:  

[Download the graph](sandbox:/mnt/data/my-image3.png)None
```

From this, we can see how the code interpreter has filtered the data to produce the required chart, which you can see below.

![](https://cdn-images-1.medium.com/max/800/1*7pwpLDCLSGAg08Qlpeg2zA.png)

### Conclusion and towards an app

Using OpenAI’s Assistants API along with the Code Interpreter allows us to generate code that will prompt GPT to create charts directly from a data file using plain English.

The code is not particularly tricky and while the Jupyter Notebook code is for demonstration purposes only, I hope that you can see how this could be easily transformed into an app that asks the user to upload a data file, enter a prompt that describes the chart required and then enables the user to download the chart as an image file.

#### Update: A prototype app

You will find a prototype Streamlit app based on the code in the GitHub repo (see below) in a folder called ‘streamlit’. To use the app, you must supply your API key and put it in a Streamlit secrets file.

The app uses the Streamlit file upload control to upload a CSV file to work with and a prompt can be entered in an input box. While the prompt is running a status string is displayed. If the LLM cannot understand your prompt or there is any other error then a simple error message is displayed.

A modified version of the Jupyter Notebook above is coded as a class in a local library package and the Streamlit app calls the methods in it. There are a couple of simple data files, too. Feel free to download and modify it and run it for your own purposes **but you must *first* read the README.md file in the ‘streamlit’ folder**!

---

Thanks for reading. You can find the code and data in a GitHub repo [here](https://github.com/alanjones2/assistantscharts2). Feel free to download, clone or fork it. To see more articles you can follow me on [Medium](https://medium.com/@alan-jones/subscribe) or subscribe to my free, occasional, [newsletter](http://technofile.substack.com). Articles are also listed on my [webpage](http://alanjones2.github.io).

*All images and screenshots are by me, the author unless otherwise noted.*

*All code is covered by the MIT license (see the repo) and, while not necessary, attribution is always appreciated.*
