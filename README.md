# Final Project using Python and Django
As my final project for Python I was asked to work on a team and develop an MVC website using Django. I was tasked to create one part of our website about a collection of things. I chose Herbs as my subject. My application is able to perform the four basic operations of CRUD. Users can add new herbs or recipes, view them, update them or delete them. They can view only one item of their choice or all of them. I also have a page that displays information sourced from another website. I used Beautiful Soup to data scrape the site and find recipes relevant to my subject. When the data scraping page loads, a list of recipes are displayed that are updated as the other site changes. Another page I have connects to the Open Library's API and by getting a JSON response, it displays borrowable books from the library on the topic of herbs and food. Besides the back end stories, I also worked on front end. I used Bootstrap, CSS and some JavaScript to create styling and add some animation. 

This was a super fun project. I absolutely love the structure of Django and the simplicity of Python. It was also a great experience to work on a team and connect on daily standup meetings.

Below are descriptions of the stories I worked on, along with code snippets.
 
## CRUD Functionality

# This function allows users to choose an herb from the home page and view information about only that herb on a separate page:
  def herbs_home(request):
     form = RecipeForm(data=request.POST or None)  # Retrieve the herb form
     # checks if request method is POST
     if request.method == 'POST':
         pk = request.POST['Herb']  # if the form is submitted, retrieve which herb
         # the user wants to view
         return herbDetails(request, pk)  # call herbDetails function
         # return requested information
     return render(request, 'herbs/herbs_home.html', {'form': form})

# This function will display details about queried herb:
def herbDetails(request, pk): #herbDetails have two arguments: request and pk which refers to the herb's primary key
    Herbinfo = Newherb.Newherbs.filter(pk=pk) #Refer back to the Newherb model and filter it by its primary key
    return render (request, 'Herbs/herbs_herbdetails.html', {'Herbinfo': Herbinfo})

# This function will allow to update an herb:
def updateHerb(request, pk):
    Uherb = Newherb.Newherbs.get(pk=pk) # Get the primary key of the requested herb from the Newherb model
    form = NewherbForm(data=request.POST or None, instance=Uherb) # Retrieve the NewherbForm
    # instance=Uherb references this particular instance of the NewherbForm, so the form can show the information
    # about the requested herb before updating it
    if request.method == 'POST': # checks if request method is POST
        if form.is_valid(): # checks if form is valid and if so, it saves it
            form.save()
            return herbDetails(request, pk) # once update is completed, return to the herbdetails page
    # pass content to the template in the dictionary
    content = {'form': form, 'Uherb': Uherb} # content variable holds a dictionary with our two variables above
    return render(request, 'Herbs/herbs_updateherb.html', content)

# This function will allow to delete an herb:
def deleteHerb(request, pk):
    Dherb=Newherb.Newherbs.filter(pk=pk) # retrieve the pk of the chosen herb from the Newherb model
    # Newherbs refers to the model manager that we specified in models.py
    if request.method == "POST": # if method is post, delete it
        Dherb.delete()
        return redirect('herbs_home') # once deleted, return to the homepage
    return render(request, 'Herbs/herbs_deleteherb.html', {'Dherb' : Dherb})

# This function allows users to see all herbs added to the database:
def allHerbs(request):
    # create a new variable named allHerbData, use objects.all() method to
    # get all the records and fields of the Newherb model.
    # objects refers to the model manager here which we named: Newherbs.
    allHerbData = Newherb.Newherbs.all().order_by('Name')
    template = loader.get_template('Herbs/herbs_allherbs.html')
    context = {
       'allHerbInfo': allHerbData,
    }
    return HttpResponse(template.render(context, request))

# This function allows users to add an herb:
def herbs_addherb(request):
    form = NewherbForm(data=request.POST or None)  # Retrieve the herb form
    # checks if request method is POST
    if request.method == 'POST':
        if form.is_valid():  # checks whether the form is valid and if so, it saves it
            form.save()  # saves new herb
            return redirect('herbs_allherbs')  # returns user back to the "all herbs" page
    # adds content of form to page
    return render(request, 'herbs/herbs_addherb.html', {'form': form})

# This function allows users to search the database for a specific herb:
def search(request):
    if request.method == "POST": # if request method is post
        searched = request.POST['searched'] # we are referring here to the searched term
        # The herbs variable holds the return search result
        # We are grabbing information from the Newherb model (from the NewherbForm) and filter by name
        # We want the name to contain whatever the person searched for, which is held in the "searched" variable
        # "Name" is a form field that we specified in the Newherb model, we can choose another form field if we want
        # if the name of the herb contains the searched term, return the result of the search
        herbs = Newherb.Newherbs.filter(Name__icontains=searched)
        return render (request, 'Herbs/herbs_search.html', {'searched': searched, 'herbs': herbs})
    else: #if not, it will leave the table empty as we didn't put anything inside of the dictionary
        return render (request, 'Herbs/herbs_search.html', {})

## Web Scraping with Beautiful Soup

# The following code is to display information sourced from another website via web scraping.

def createBeautifulsoup(request):
    url = "https://plantyou.com/category/all-recipes/" #this is the url that I'm scraping
    response = requests.get(url) # Make a get request with the requests.get() method, it requires one argument
    soup = BeautifulSoup(response.content, "html.parser") #we create a BeautifulSoup object by passing two arguments
    # BeautifulSoup parses through the HTML
    # response.content is the raw HTMl content
    # The soup object contains all the data in the nested structure - you can make it more readable by using: soup.prettify()

    # Titles of latest recipes
    all_recipes = {
        'recipe1': soup.find_all(class_='title')[0].get_text(), #To display the text inside an HTML element, we use .get_text()
        'recipe2': soup.find_all(class_='title')[1].get_text(),
        'recipe3': soup.find_all(class_='title')[2].get_text(),
        'recipe4': soup.find_all(class_='title')[3].get_text(),
        'recipe5': soup.find_all(class_='title')[4].get_text(),
        'recipe6': soup.find_all(class_='title')[5].get_text(),
        'recipe7': soup.find_all(class_='title')[6].get_text(),
        'recipe8': soup.find_all(class_='title')[7].get_text(),
        'recipe9': soup.find_all(class_='title')[8].get_text(),
        'recipe10': soup.find_all(class_='title')[9].get_text(),
    }
    # Urls of latest recipes
    all_urls = {
        'url1': soup.find_all(class_='item archive-post')[0].a['href'], #To get the link we use .a['href']
        'url2': soup.find_all(class_='item archive-post')[1].a['href'],
        'url3': soup.find_all(class_='item archive-post')[2].a['href'],
        'url4': soup.find_all(class_='item archive-post')[3].a['href'],
        'url5': soup.find_all(class_='item archive-post')[4].a['href'],
        'url6': soup.find_all(class_='item archive-post')[5].a['href'],
        'url7': soup.find_all(class_='item archive-post')[6].a['href'],
        'url8': soup.find_all(class_='item archive-post')[7].a['href'],
        'url9': soup.find_all(class_='item archive-post')[8].a['href'],
        'url10': soup.find_all(class_='item archive-post')[9].a['href'],
    }
    # put it all into the variable 'context'
    context = {'all_recipes': all_recipes, 'all_urls': all_urls}
    # display it via the .html page specified below, that is where the structure of the page is created
    return render(request, "Herbs/herbs_BeautifulSoup.html

## Connecting to API and getting a JSON response

# The following code is to display information from another website via connecting to that site's API.

def createAPI(request):
    # The requests method is used to connect to the API to extract the data from the url in the brackets
    response = requests.get("https://openlibrary.org/search.json?q=herbs.food&mode=ebooks&has_fulltext=true")
    data = response.json() # response.json() returns a JSON object of the result and we store the content in the data variable

    # Once we determined what data we want to extract, we access it.
    # In this case, the title of a book is stored in the "docs" dictionary, which has a list inside of it
    # I want to access the first item from that list (so I use [0]), which is another dictionary
    # From there I would like to access the key that is named 'title'
    # I extract the number of titles I would like to display on my webpage and store them in the titles dictionary
    titles = {
        'title1': data['docs'][0]['title'],
        'title2': data['docs'][1]['title'],
        'title3': data['docs'][2]['title'],
        'title4': data['docs'][3]['title'],
        'title5': data['docs'][4]['title'],
        'title6': data['docs'][5]['title'],
        'title7': data['docs'][6]['title'],
        'title8': data['docs'][7]['title'],
        'title9': data['docs'][8]['title'],
        'title10': data['docs'][9]['title'],
        'title11': data['docs'][10]['title'],
        'title12': data['docs'][11]['title'],
        'title13': data['docs'][12]['title'],
        'title14': data['docs'][13]['title'],
        'title15': data['docs'][14]['title'],
    }

    # I do the same for the authors dictionary as for the titles
    # In this case, author_name is a list, and I'm displaying only the first item of that list, so I'm using [0]
    authors = {
        'author1': data['docs'][0]['author_name'][0],
        'author2': data['docs'][1]['author_name'][0],
        'author3': data['docs'][2]['author_name'][0],
        'author4': data['docs'][3]['author_name'][0],
        'author5': data['docs'][4]['author_name'][0],
        'author6': data['docs'][5]['author_name'][0],
        'author7': data['docs'][6]['author_name'][0],
        'author8': data['docs'][7]['author_name'][0],
        'author9': data['docs'][8]['author_name'][0],
        'author10': data['docs'][9]['author_name'][0],
        'author11': data['docs'][10]['author_name'][0],
        'author12': data['docs'][11]['author_name'][0],
        'author13': data['docs'][12]['author_name'][0],
        'author14': data['docs'][13]['author_name'][0],
        'author15': data['docs'][14]['author_name'][0],
    }

    year = {
        'year1': data['docs'][0]['first_publish_year'],
        'year2': data['docs'][1]['first_publish_year'],
        'year3': data['docs'][2]['first_publish_year'],
        'year4': data['docs'][3]['first_publish_year'],
        'year5': data['docs'][4]['first_publish_year'],
        'year6': data['docs'][5]['first_publish_year'],
        'year7': data['docs'][6]['first_publish_year'],
        'year8': data['docs'][7]['first_publish_year'],
        'year9': data['docs'][8]['first_publish_year'],
        'year10': data['docs'][9]['first_publish_year'],
        'year11': data['docs'][10]['first_publish_year'],
        'year12': data['docs'][11]['first_publish_year'],
        'year13': data['docs'][12]['first_publish_year'],
        'year14': data['docs'][13]['first_publish_year'],
        'year15': data['docs'][14]['first_publish_year'],
    }

    # I store each dictionary I want to use in the context variable
    context = {'titles': titles, 'authors': authors, 'year': year}

    # I return the results on the herbs_API page
    return render(request, "Herbs/herbs_API.html", context)

