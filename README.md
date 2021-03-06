
<h2>Surf Spots Python Internship App</h2>
  This was an intership experience  with Prosper I.T. Consulting. This project was a great experience for me using Agile/Scrum methodologies during a  two week sprint. I created a Surf Spots  App as part of a much larger Hobby Tracker website and created a database for the App to connect with and log favorite surf spots with full CRUD functionality.
This included using Python3 in virtual environments, Pycharm as the IDE and Git as version control..<br>
<br>
Here is a code example of my views for this project:
<br>
 Home page ------<br>
 
![SurfSpotsAppHome](https://user-images.githubusercontent.com/68976585/103727518-50896a80-4f90-11eb-89ce-df33eeeb3dde.png)
<br>
This is the Home page View request that opens the home page as well as displays an current list of saved spots on it. 
```
def surfApp_home(request):
# get the list of spotNames to show your list of favorite spots on the home page
    spots = SurfSpot.objects.all()
    context = {'spots': spots}
#link to home page and display requested content
    return render(request, 'surfApp/surfApp_home.html', context)
```    
 Add Spot page ------ use model to create a form and post to database
 ![SurfSpotAppCreate](https://user-images.githubusercontent.com/68976585/103727616-8e868e80-4f90-11eb-91bc-8ca284f71b18.png)
 This is the Add your spots page that will display a surf spot form and if filled out properly, will save it to the database 
 and redirect to show the new spot on the index page. If the form is not filled out properly it will request necessary fields 
 to be filled before continuing or canceling.
 
```
def surfApp_addSpot(request):
# gets requested form if exists
    form = SurfSpotForm(request.POST or None)
    if form.is_valid():
        #  checks that form has the required entries filled in and if
        #  valid with no errors, saves to the database
        form.save()
        #when done with saving form, this redirects to the index (list of spots) page
        return redirect('SpotIndex')

    else:
# if not saved, print errors to the terminal
        print(form.errors)
        form = SurfSpotForm()
    context = {'form': form, }
    return render(request, 'surfApp/surfApp_addSpot.html',  context)
```
My model for this form:

```
from django.db import models
#  rating_choices for ratings using an integer for the field
RATING_CHOICES = [
    (1, 1),
    (2, 2),
    (3, 3),
    (4, 4),
    (5, 5),
]
class SurfSpot(models.Model):
    spotName = models.CharField(max_length=50, verbose_name="Surf Spot Name", default="", null=False)
    location = models.CharField(max_length=80, verbose_name="County/State", default="", null=False)
    description = models.TextField(max_length=300, verbose_name="Your Description", default="", blank=True)
    rating = models.IntegerField(verbose_name="Your Rating", choices=RATING_CHOICES, null=False)

    objects = models.Manager()

    def __str__(self):
        return self.spotName
```
 Index page ------ 
 ![SurfSpotAppIndex](https://user-images.githubusercontent.com/68976585/103727699-c1308700-4f90-11eb-89da-e3565d74c35f.png)
This is the Index page that gets all saved spots from the database as dictionary items to render as a list, 
each spot with a link to its own details page. 
```
def surfApp_index(request):
# gets all posts from database
    spots = SurfSpot.objects.all()
#print to terminal to make sure data is populating
    print(spots)
#creates dictionary for items in the database and passes the args to the page
    context = {'spots': spots, }
    return render(request, 'surfApp/surfApp_index.html', context)
```
Here in my Index Template, using Django syntax, is the anchor tag with the link to each 'SpotDetails'.
```
{% block content %}

{% for spot in spots %}
<!-- passing an argument or variable through url (had a no reverse match error)-->
<a href="{% url 'SpotDetails' spot.pk %}" ><h3>{{ spot.spotName | upper }} ~ {{ spot.location | title }} ~ Rating: {{ spot.rating }}</h3></a>

{% endfor %}

{% endblock %}
```
  Details page ------ use primary key to get details from that item
  ![SurfSpotAppDetails](https://user-images.githubusercontent.com/68976585/103727727-d5748400-4f90-11eb-8a85-52037cb052d5.png)
This is the details page that uses a primary key to get details from the database for that item as well as the ablity to update or delete the spot.
```
def surfApp_details(request, pk):
# gets a single instance of the SurfSpot model object from the database
    spot = get_object_or_404(SurfSpot, pk=pk)
    context = {'spot': spot, }
    return render(request, 'surfApp/surfApp_details.html', context)
```
 Update page (edit) ------Pulls up populated form from database to be updated and reposted to database.
```
def surfApp_update(request, pk):
# means Django expects an integer value and will transfer it to a view as a variable called pk
    pk = int(pk)
    spot = get_object_or_404(SurfSpot, pk=pk)
 HTTP method POST-- finds the form that was submitted by a user
    if request.method == 'POST':
        # and we can find that forms filled out answers with the request.POST
        form = SurfSpotForm(request.POST, instance=spot)
        if form.is_valid():
            form.save()
            return redirect('SpotIndex')
        else:
            print(form.errors)
    form = SurfSpotForm(instance=spot)
    context = {'form': form, }
    return render(request, 'surfApp/surfApp_update.html', context)
```
 Delete ------ from details page, get instance of spot and redirect to confirm delete message page.
```
def surfApp_delete(request, pk):
    pk = int(pk)
    spot = get_object_or_404(SurfSpot, pk=pk)
    context = {'spot': spot, }
    return render(request, 'surfApp/surfApp_delete.html', context)
```
 Confirm delete ------ renders delete page for confirmation of action
 ![SurfSpotsAppDelete](https://user-images.githubusercontent.com/68976585/103727794-0b196d00-4f91-11eb-8b98-6f1a1667c5f8.png)
```
def confirm_delete(request, pk):
    pk = int(pk)
    spot = get_object_or_404(SurfSpot, pk=pk)
    spot.delete()
    return redirect('SurfSpotsHome')
```
<strong>And my urlpatterns for this app:</strong>
```
urlpatterns = [
    path('', views.surfApp_home, name='SurfSpotsHome'),
    path('AddSpot', views.surfApp_addSpot, name='AddSpot'),
    path('SpotIndex', views.surfApp_index, name='SpotIndex'),
    path('<int:pk>/SpotDetails', views.surfApp_details, name='SpotDetails'),
    path('<int:pk>/UpdateSpot', views.surfApp_update, name='UpdateSpot'),
    path('<int:pk>/DeleteSpot', views.surfApp_delete, name='DeleteSpot'),
    path('<int:pk>/ConfirmDelete', views.confirm_delete, name='ConfirmDelete'),
]
```
<h2>Conclusion</h2>
I enjoyed creating something I would like to use and is close to me personally. I learned a lot about how to, and why, working in virtual environments is such an important process to master. What a great experience to work along side other developers working on their own projects within the main Hobby Tracker app and seeing how many ways to solve a single problem there are. I learned a lot about version control in PyCharm and I liked how easy it became to use once versed. In retrospect, I would have liked to have been able to complete the API/webscraping stories and have each spot be able to get current surf conditions for the day or hour. 
