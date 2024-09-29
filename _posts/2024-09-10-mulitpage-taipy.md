---
layout: post
image: https://github.com/alanjones2/taipyapps/blob/main/multipage/images/ria-CvTaPeo3NRk-unsplash.jpg?raw=true
title: How to Construct a Multipage Data Science Web App in Python with Taipy
excerpt: Taipy supports easy navigation between pages in Python web apps - we create a simple CO2 emissions app
---


![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/ria-CvTaPeo3NRk-unsplash.jpg?raw=true)

*Photo by [Ria](https://unsplash.com/@riapuskas?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/factory-chimney-emitting-smoke-CvTaPeo3NRk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Taipy is a framework for building Data Science and AI web apps in Python. As such, it is a competitor to the likes of Streamlit or Dash, but it is distinct from both of those products. 

Taipy separates the user interface from the rest of the program logic and uses callbacks to add functionality to user controls. In this sense, it is closer to Dash than Streamlit where the user interface controls are often incorporated into the main Python code. 

Both Dash and Taipy are based on the Flask microframework, so it shouldn't be a surprise that there are similarities but where in a Dash app, you essentially construct your user interface in HTML (but written with Python functions), Taipy has an additional layer of abstraction that lets the user define user controls that are closer to Streamlit than Dash.

So, is Taipy the best of both worlds?  I wrote an introduction in [A Data Dashboard in Pure Python with Taipy](https://medium.com/towards-data-science/a-data-dashboard-in-pure-python-with-taipy-bdb164a62b8b) that explores how to construct a web app in Taipy, so you can judge for yourselves how easy it is to use.

This article goes a step further than the first by looking at how multipage apps can be constructed with Taipy but first let's spend a minute constructing a simple one-page app (it's only three lines of code!)

If you want to code along, you will need a Python installation, 3.8 or newer, and you will need to...

```bash
pip install taipy
```

Of course!

### Hello World the Taipy way

The basic building block for a Taipy web app is a page. You define it in Python, Markdown or HTML (your choice) and then add the application logic in Python linking it to the user interface using callback functions.

Here is the code for the ubiquitous 'Hello World' app.

```python
from taipy.gui import Gui

page = "## Hello World"

Gui(page=page).run()
```

As you can probably deduce, a single page is defined by a Markdown string, `page` and the app is started by passing that page to the `Gui()` function. (In a real application, the page would likely include controls for user input and callback functions written in Python linked to those controls.)

You run the app as a normal Python application (file names are as per my GitHub repo - see note [2]):

```bash
python example0.py
```

And it will be available in your browser on your local host: http://127.0.0.1:5000/

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/Screenshot_example0.png?raw=true)

Easy.

### Multiple pages

To display multiple pages it is simply a matter of defining those pages individually and giving each page a named endpoint. Navigation between the pages can be done explicitly, using the `navigate()` function or a built-in navigation bar.

Shortly, we will build a multipage app and explore how to construct the pages and navigate between them using both methods. This app will define the pages in Python and will feature interactive charts and callbacks linked to the UI controls.

But to illustrate the basic structure, here is the code for a simple two-page app that defines the pages as Markdown strings.

```python
from taipy.gui import Gui

page1 = "## This is page 1"
page2 = "## This is page 2"

pages = {
    "page1": page1,
    "page2": page2,
}

Gui(pages=pages).run()
```

The actual pages are defined in the strings `page1` and `page2` and display the Markdown text in those strings. The pages are grouped in a dictionary along with their endpoint names and these are then passed to the function `Gui` which runs the app.

This code is in a file called _example1.py_ so we run it, thus:

```bash
python example1.py
```

The pages are named "page1" and "page2" and when you run the app locally "page1" is accessible on the local server: http://127.0.0.1:5000/page1 and "page2" on http://127.0.0.1:5000/page2.

Ideally, of course, you want to have some sort of navigation between the pages. A simple way of doing this is to include a navigation bar.

### Navigation bar

Taipy provides a navigation bar control that you can use in a Markdown page definition. To use it as a common navigation bar across all pages, we need to define a third page which maps onto the root endpoint (`'/'`).

You can see this in the code below. There is a new page called `root` which contains the navigation bar code. The other two pages are as before and the dictionary now contains a new entry for the root page.

```python
from taipy.gui import Gui

root = """
<|navbar|>  
"""
page1 = "## This is page 1"
page2 = "## This is page 2"

pages = {
    "/": root,
    "page1": page1,
    "page2": page2,
}

Gui(pages=pages).run()
```

Now run:

```bash
python example2.py
```

Navigate to http://127.0.0.1:5000/, and you will see that the navigation bar is displayed at the top of the screen and the browser will automatically navigate to the first page. You can then use the navigation bar to move from one page to the other. In the image below I have selected PAGE2.

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/Screenshot_example2_dark.png?raw=true)

Taipy has created a sort of template for us where the content of the root endpoint is displayed as a header above all of the other pages.

What if we want a footer, too?

To create a website where the content of each page has a common header and footer we use the pseudo control `<|content|>` in the root page. This pseudo control explicitly tells Taipy where the content of the web page should be located (in the previous example it was implicitly located after the header).

In the code below we see that the `root` page now contains the content pseudo control and it is placed immediately after the navigation bar. By doing this we can now add a footer to each page by simply adding more Markdown code after the content. In this case, the footer contains the text, _"This is a footer"_.

```python
from taipy.gui import Gui

root = """
<|navbar|>  
<|content|>
_This is a footer_
"""

page1 = "## This is page 1"
page2 = "## This is page 2"

pages = {
    "/": root,
    "page1": page1,
    "page2": page2,
}

Gui(pages=pages).run()
```

You can see the result in the image below.

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/Screenshot_example3_dark.png?raw=true)

Selecting page 2 would show the contents of page 2 with the same header and footer.

### The multipage app

Let's look at something a bit more interesting. 

We are going to develop a simple three-page app that explores CO2 emissions by country and by the source of the emissions.

The app consists of a home page that gives us a summary of the app and ways to navigate to the other two pages,

You can see a screenshot of the home page, below. 

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/appscreenshot.png?raw=true)

It is a simple app that consists of three pages: the home screen plus two pages that contain interactive graphs that explore sources of global CO2 emissions and a map of CO2 emissions by country. The graphs are drawn using Plotly.

At the top of the screen is the navigation bar that links to the three pages that comprise the app, while, below the images on the home screen, are buttons that provide an alternative way of navigating between the pages.

The other pages are shown below. 

The first is a bar graph of the CO2 emissions between 1850 and 2020 - you can show the total emissions or those from a particular source using the dropdown menu.

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/co2emissionspagescreenshot.png?raw=true)

The second one is a choropleth showing annual CO2 emissions by country for a range of years.

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/worldtemppagescreenshot.png?raw=true)

You can see how the header (navigation bar) and the footer are preserved in each page.

Of course, these pages are more complicated than the simple strings that we have seen so far. So, to make things neat and easy to follow, we will define each page as a separate entity, we'll define them in Python and we will make each page a package (i.e. each page will be in a separate Python file that we can import into the main program).

The files for the pages will be called *home.py*, *CO2_src_chart.py* and *CO2_map.py*. In each of the page files we will define a variable `page` that will contain the page definition and those variables will be imported into the main program. In addition, there will be a main program *main.py*.

Let's take a look at _main.py_.

```python
from taipy.gui import Gui
from pages.home import page as homepage
from pages.CO2_src_chart import page as co2page
from pages.CO2_map import page as CO2countrypage

root_md = """
<|navbar|>
<|content|>  
_Made with Taipy_    
"""

pages = {
    "/": root_md,
    "home": homepage,
    "CO2_Sources": co2page,
    "CO2_by_Country":CO2countrypage,
}

Gui(pages=pages).run(dark_mode=False)
```

As you can see it follows the same pattern as the simple apps that we saw before.  The main difference is that we import the pages from external files rather than defining them as strings. The root endpoint _is_ defined as a Markdown string and is almost identical to the earlier ones.

Running this file will give us a webpage with a navigation bar and a footer that says "*Made with Taipy*".

There is one more minor addition. In the call to the  `run` function (the last line), we set a parameter `dark_mode` to `False`. By default, Taipy uses dark mode for the display and this is the way to change it. I have nothing against dark mode but I think the lighter colours go better with the default Plotly theme in this app.

## The pages

I discussed creating webpages in Python with Taipy in [A Data Dashboard in Pure Python with Taipy](https://medium.com/towards-data-science/a-data-dashboard-in-pure-python-with-taipy-bdb164a62b8b) so I'll try not to repeat myself too much here.

Defining pages as separate Python packages is very convenient and reminiscent of the way you design a multipage Streamlit app. Although, unlike Streamlit, you can locate the files wherever you want to; here we put them in a folder called _pages_. Let's look at the simplest page first.

### CO2 sources page

```python
import taipy.gui.builder as tgb
import pandas as pd
import plotly.express as px

def plot_chart(df, y):
    fig = px.bar(df, x="Year", y=y)
    return fig

def on_select(state, var, val):
    state.fig2 = plot_chart(state.df, y=val)

df = pd.read_csv("data/co2-emissions-by-category.csv") 
src = "Total"
fig2 = plot_chart(df, src)

with tgb.Page() as page:
    tgb.text( value = "### CO2 Emissions by source since the mid-19th Century", mode = 'md', class_name="color-secondary")
    tgb.selector(label="Select a CO2 Source", 
                    value="{src}", 
                    lov="Total;Coal;Oil;Gas;Cement", 
                    dropdown=True, 
                    on_change=on_select)
    tgb.chart(figure="{fig2}")


```

This page displays the bar chart of CO2 emissions by source. To select the source it utilises a drop-down menu, the chart is drawn using Plotly and the data, derived from Our World in Data[1], are read from a CSV file. The page itself is constructed using Taipy's GUI Builder package.

The imports are self-explanatory and, for the moment, let's ignore the function definitions. The first code to be executed sets up some state (global) variables that will be used to create the default page: 

- `df` is a Pandas dataframe with the emissions data

- `src` is a string - the default CO2 source

- `fig2` is set to an initial bar chart
  
  

![](https://github.com/alanjones2/taipyapps/blob/main/multipage/images/co2sourcedata.png?raw=true)

You can see a snapshot of the data above. In the bar chart, we will be displaying the total emissions or those sourced from coal, oil, gas, and cement over all of the years covered in the data.

Let's see how we construct the page.

We use `tgb.Page()` to create a variable called `page` and  use a `with` statement to create the elements within the page.

The first element is a simple header in a Markdown string and the last element is a chart which takes the state variable `fig2` as a parameter (the curly brace notation binds the state variable `fig2` to the chart element so that whenever `fig2` is updated the chart element is updated, too).

This leaves the bit in the middle which is a call to `tgb.selector()`  -  this implements the drop-down menu.

```python
    tgb.selector(label="Select a CO2 Source", 
                    value="{src}", 
                    lov="Total;Coal;Oil;Gas;Cement", 
                    dropdown=True, 
                    on_change=on_select)
```

The parameter values are:

-  a label for the menu,

- the default menu selection,

- the list of variables that will appear in the menu "Total;Coal;Oil;Gas;Cement",

- a flag to determine whether the menu drops down (otherwise it is a fixed menu),

- the callback function that will be called when the menu selection changes.

So the default selection is "Total" (we initialised it earlier) and when we change this the function `on_select()` is called.

```python
def on_select(state, var, val):
    state.fig2 = plot_chart(state.df, y=val)
```

All callback functions have the same default parameters: 

`state`: is the set of state (global) variables,

`var`: is the name of the variable that is bound to the called control,

`val`: is the value of `var`.

And we use these to call the plotting function and update the state variable `fig2` which, because it is bound to the chart element on the page, means that the chart element is updated with the chart for the newly selected CO2 source.

### CO2 Map page

This page looks more complex the the previous one but that is mainly due to the extra code needed to create a choropleth.

```python
import taipy.gui.builder as tgb
import pandas as pd
import plotly.express as px

year_max = 2020
year_min = 1950
year = year_max

# define parameters for map graphic
df_total = pd.read_csv('data/co2_total.csv')
col = 'Annual CO₂ emissions'    # the column that contains the emissions data
max = df_total[col].max()       # maximum emissions value for color range
min = df_total[col].min()       # minimum emissions value for color range

def plot_choro(year):
    fig = px.choropleth(df_total[df_total['Year']==year], 
        locations="Code",       # The ISO code for the Entity (country)
        color=col,              # color is set by this column
        hover_name="Entity",    # hover name is the name of the Entity (country)
        range_color=(min,max),  # the range of values as set above
        scope= 'world',         # a world map - the default
        projection='equirectangular', 
        title=f"CO2 Emissions by country in {year}",
        color_continuous_scale=px.colors.sequential.Reds
        )

    fig.update_layout(
        margin=dict(l=0, r=0, t=0, b=0),
        )

    return fig

fig = plot_choro(year_max)

def on_slider(state, var, val):    
    state.fig = plot_choro(val)

with tgb.Page() as page:
    tgb.text( value= f"### Country CO2 Emissions from {year_min} to {year_max}", mode='md', class_name="color-secondary")
    tgb.text(value="#### Use the slider to select a year", mode='md')
    tgb.slider(value="{year}", min=year_min, max=year_max, on_change=on_slider)
    tgb.chart(figure="{fig}")

```

If you look closely you will see that the structure is very similar to the *CO2 Sources* page. The main difference to the page layout is that we include a slider element instead of a menu. The remainder is much the same: some state variables are given initial values; there is a plotting function; and there is a callback function that calls the plotting function in the same way as before.

The more interesting page is the home page as this uses the `navigate()` function to implement navigation separately from the menu bar.

### The Home page with button navigation

Here we demonstrate the second method of navigation. We still have the navigation bar (that's on all of the pages) but here we also see the use of the `navigate()` function.

```python
from taipy.gui import navigate
import taipy.gui.builder as tgb

def on_button_press(state, id):
    page = "home"
    if id == "CO2": page = "CO2_Sources" 
    if id == "CO2Country": page = "CO2_by_Country"
    navigate(state, to=page)

with tgb.Page() as page:
    tgb.text( value = "## World CO2 Emissions", mode = 'md', class_name="color-secondary")
    tgb.text( value = "#### Explore the graphs using the navigation bar above, or the buttons below.", 
             mode = 'md')
    with tgb.layout(columns="1 1"):
        with tgb.part():
            tgb.text("![](images/co2emissionspagesmall.png)", mode='md') 
            with tgb.layout("2 1"):
                with tgb.part():
                    tgb.text("__Select the Co2 Emissions page here:__", mode='md')
                with tgb.part():
                    tgb.button("CO2 by source", id="CO2",  on_action=on_button_press)
        with tgb.part():
            tgb.text("![](images/worldtemppagesmall.png)", mode='md') 
            with tgb.layout("2 1"):
                with tgb.part():
                    tgb.text("__Select CO2 by Country page here:__", mode='md')
                with tgb.part():
                    tgb.button("CO2 by Country", id="CO2Country", on_action=on_button_press)

```

The page begins with text headers and then is split into two columns - each column is defined in a `tgb.part()`. In those columns we display images of the other pages via Markdown text and then, below the images there are two more columns, the left one for a label and the right one for a button. The callback function defined for the two buttons is the same. `on_button_press()` and each button had an `id `set for it - this id will be passed into the callback function.

The callback, itself, looks at the `id `and determines which page to navigate to using the `navigate()` function, which, as you can see, requires the state and a page name to be passed to it.

And that's it.

## Conclusion

We've seen how easy it is to create a multipage app in Taipy by creating each page in a separate file, setting the content to a local variable `page `and then importing each of the pages into a main program.

Navigation can be automatically included using a navigation bar control or by explicit use of the `navigation `function.

Taipy, an interesting alternative to Dash or Streamlit, has a character and capabilities of its own. It is not such a mature product as the other two, perhaps, but looks very capable.

---

Thanks for reading and I hope you enjoyed this look at multipage apps in Taipy. Please download the code and data (link below) and give it a go. Any opinions, criticisms or any other comments are always welcome.

To see more articles, follow me here on [Medium](https://medium.com/@alan-jones/subscribe), or subscribe to my free, occasional, [newsletter](http://technofile.substack.com/). Older articles are listed on my [webpage](http://alanjones2.github.io/).

# Notes

1. The data used in this article is derived from [Our World in Data](https://ourworldindata.org/). OWD publishes articles and data about the most pressing problems that the world faces. All its content is open source and its data is downloadable — see [About — Our World in Data](https://ourworldindata.org/about) for details.
2. Code and data for this app are available in my [GitHub](https://github.com/alanjones2/taipyapps) repo in the folder ‘multipage’.
3. All images, screenshots, diagrams, etc. are by me, the author, unless otherwise noted.
4. While there is a commercial version of Taipy, this article uses the open source version.
5. Disclaimer: I have no connection with Taipy other than as a user of their products.