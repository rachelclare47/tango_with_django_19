#User Authentication

The aim of this next part of the tutorial is to get you familiar with
the user authentication mechanisms provided by Django. We'll be using
the `auth` application provided as part of a standard Django
installation in package `django.contrib.auth`. According to [Django's
official documentation on
Authentication](https://docs.djangoproject.com/en/1.9/topics/auth/), the
application consists of the following aspects.

- *Users.*
- *Permissions:* a series of binary flags (e.g. yes/no) determining what a user may or may not do.
- *Groups:* a method of applying permissions to more than one user.
- A configurable *password hashing system:* a must for ensuring data security.
- *Forms and view tools for logging in users,* or restricting content.

There's lots that Django can do for you in the area of user
authentication. We'll be covering the basics to get you started. This
will help you build your confidence with the available tools and their
underlying concepts.

##Setting up Authentication

Before you can begin to play around with Django's authentication
offering, you'll need to make sure that the relevant settings are
present in your Rango project's `settings.py` file.

Within the `settings.py` file find the `INSTALLED_APPS` list and check
that `django.contrib.auth` and `django.contrib.contenttypes` are listed,
so that it looks like the code below:

{lang="python",linenos=off}
	INSTALLED_APPS =[
		'django.contrib.admin',
		'django.contrib.auth',
		'django.contrib.contenttypes',
		'django.contrib.sessions',
		'django.contrib.messages',
		'django.contrib.staticfiles',
		'rango',
	]

While `django.contrib.auth` provides Django with access to the
authentication system, `django.contrib.contenttypes` is used by the
authentication application to track models installed in your database.

I> ### Migrate, if necessary!
I>
I> Remember, if you had to add `django.contrib.auth` and `django.contrib.contenttypes` applications to your
I> `INSTALLED_APPS` tuple, you will need to update your database with the
I> `$ python manage.py migrate` command.


## Password Hashing
Passwords are stored by default in Django using the [PBKDF2
algorithm](http://en.wikipedia.org/wiki/PBKDF2), providing a good level
of security for your user's data.  However, if you want more control over how the passwords are hashed, then in the `settings.py` add in tuple to specify the `PASSWORD_HASHERS`:

{lang="python",linenos=off}
	PASSWORD_HASHERS = (
		'django.contrib.auth.hashers.PBKDF2PasswordHasher',
		'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
	)

Django will use the first hasher in `PASSWORD_HASHERS` i.e.
`settings.PASSWORD\_HASHERS[0]`. If other password hashers are listed it will also support these if the a match is not obtained using the first one.


If you want to use a more secure
hasher, you can install [Bcrypt](https://pypi.python.org/pypi/bcrypt/) using `pip install bcrypt`, and
then set the `PASSWORD_HASHERS` to be:

{lang="python",linenos=off}
	PASSWORD_HASHERS = [
		'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
		'django.contrib.auth.hashers.BCryptPasswordHasher',
		'django.contrib.auth.hashers.PBKDF2PasswordHasher',
		'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
	]

If you don't explicitly specify 
`PASSWORD_HASHERS`, Django will use
`django.contrib.auth.hashers.PBKDF2PasswordHasher` by default.

You can read more about password hashing in the [official Django documentation on how Django stores
passwords](https://docs.djangoproject.com/en/1.9/topics/auth/passwords/#how-django-stores-passwords).



## Password Validators
A new feature introduced to Django 1.9 is [password validation](https://docs.djangoproject.com/en/1.9/topics/auth/passwords/#password-validation). In `settings.py`, you will notice a list of dictionaries called, `AUTH_PASSWORD_VALIDATORS`. As you can see Django comes with a number of default password validators for common checks such as length. The different validators can be configured, for example, if you wanted to ensure passwords are at least 6 in length, you can set `min_length` parameter of the `MinimumLengthValidator` as follows:

{lang="python",linenos=off}
	AUTH_PASSWORD_VALIDATORS = [
	
	...
	
	{
		'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
			'OPTIONS': { 'min_length': 6, }
	},
	
	...
	
	]
	
It is also possible to create your own password validators. For more information see the [official Django documentation on password validators](https://docs.djangoproject.com/en/1.9/topics/auth/passwords/#password-validation).


##The `User` Model

The core of Django's authentication system is the `User` object, located
at `django.contrib.auth.models.User`. A `User` object represents each of
the people interacting with a Django application. The [Django
documentation on User
objects](https://docs.djangoproject.com/en/1.9/topics/auth/default/#user-objects)
states that they are used to allow aspects of the authentication system
like access restriction, registration of new user profiles and the
association of creators with site content.

The `User` model has five main fields. They are:

-   the username for the user account;
-   the account's password;
-   the user's email address;
-   the user's first name; and
-   the user's surname.

The model also comes with other attributes such as `is_active`, `is_staff` , `is_superuser` which are Boolean fields to denoted whether the account is active, is owned by a staff member or has superuser privileges, respectively.  Check the
[official Django documentation on the user
model](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User)
for a full list of attributes provided by the base `User` model.

##Additional User Attributes
If you would like to include other user related attributes than what is provided by
the `User` model, then you will needed to create a model that is
associated with the the `User` model. For our Rango application, we want
to include two more additional attributes for each user account.
Specifically, we wish to include:

- a `URLField`, allowing a user of Rango to specify their own website; and
- a `ImageField`, which allows users to specify a picture for their user profile.

This can be achieved by creating an additional model in Rango's
`models.py` file. Let's add a new model called `UserProfile`:

{lang="python",linenos=off}
	class UserProfile(models.Model):
		# This line is required. Links UserProfile to a User model instance.
		user = models.OneToOneField(User)
		
		# The additional attributes we wish to include.
		website = models.URLField(blank=True)
		picture = models.ImageField(upload_to='profile_images', blank=True)
		
		# Override the __unicode__() method to return out something meaningful!
		def __unicode__(self):
			return self.user.username

Note that we reference the `User` model using a one-to-one relationship.
Since we reference the default `User` model, we need to import it within
the `models.py` file:

{lang="python",linenos=off}
	from django.contrib.auth.models import User

It may have been tempting to add these additional fields by inheriting
from the `User` model directly. However, because other applications may
also want access to the `User` model, then it not recommended to use
inheritance, but instead use the one-to-one relationship.

For Rango, we've added two fields to complete our user profile, and
provided a `__unicode__()` method to return a meaningful value when a
unicode representation of a `UserProfile` model instance is requested.

For the two fields `website` and `picture`, we have set `blank=True` for
both. This allows each of the fields to be blank if necessary, meaning
that users do not have to supply values for the attributes if they do
not wish to.

Also note that the `ImageField` field has an `upload_to` attribute. The value
of this attribute is conjoined with the project's `MEDIA_ROOT` setting
to provide a path with which uploaded profile images will be stored. For
example, a `MEDIA_ROOT` of
`<workspace>/tango_with_django_project/media/` and `upload_to` attribute
of `profile_images` will result in all profile images being stored in
the directory
`<workspace>/tango_with_django_project/media/profile_images/`.

I> ###Take the Pil
I>
I> The Django `ImageField` field makes use of the *Python Imaging Library
I> (PIL)*. If you have not done so already, install PIL via Pip, `pip install pillow` or `pip install pillow    --global-option="build_ext" --global-option="--disable-jpeg"` if you have `jpeg` support.
I> You can check what packages are installed in your (virtual) environment by issuing  the command, `pip list`.

To make the `UserProfile` model data accessible via the Django admin web interface add the following lines to the `admin.py` file.

{lang="python",linenos=off}
	from rango.models import UserProfile
	
	...
	
	admin.site.register(UserProfile)


I> ###Migrate
I>
I> Remember that your database must be updated with the creation of a new
I> model. Run `$ python manage.py makemigrations rango` from your
I> terminal to create the migration scripts for the new `UserProfile`
I> model. Then run `$ python manage.py migrate`

##Creating a *User Registration* View and Template

With our authentication infrastructure laid out, we can now begin to
build onto it by providing users of our application with the opportunity
to create user accounts. We can achieve this by creating a new view, template and URL mapping to handle user registration.


I> ###Django User Registration Applications
I>
I> It is important to note that there are several off-the-shelf user
I> registration applications available which reduce a lot of the hassle of
I> building your own registration and log in forms. 
I> 
I> However it is good idea to get a feeling for the underlying mechanics before using such
I> applications so that you have some sense of what is going on under the hood.
I> It will also re-enforce your understanding of working
I> with forms, how to extend upon the user model, and how to upload media (which can cause a lot of grief).

To provide the user registration functionality we will go through the
following steps:

1.  Create a `UserForm` and `UserProfileForm`.
2.  Add a view to handle the creation of a new user.
3.  Create a template that displays the `UserForm` and `UserProfileForm`.
4.  Map a URL to the view created.
5.  Link the index page to the register page

### Creating the `UserForm` and `UserProfileForm`

In `rango/forms.py`, we now need to create two classes inheriting from
`forms.ModelForm`. We'll be creating one for the base `User` class, as
well as one for the new `UserProfile` model that we just created. The
two `ModelForm` inheriting classes allow us to display a HTML form
displaying the necessary form fields for a particular model, taking away
a significant amount of work for us. Neat!

In `rango/forms.py`, let's create our two classes which inherit from
`forms.ModelForm`. Add the following code to the module.

{lang="python",linenos=off}
	class UserForm(forms.ModelForm):
	password = forms.CharField(widget=forms.PasswordInput())

	class Meta:
		model = User
		fields = ('username', 'email', 'password')
		
	class UserProfileForm(forms.ModelForm):
	class Meta:
		model = UserProfile
		fields = ('website', 'picture')


You'll notice that within both classes, we added a
[nested](http://www.brpreiss.com/books/opus7/html/page598.html) `Meta`
class. As [the name of the nested class
suggests](http://www.webopedia.com/TERM/M/meta.html), anything within a
nested `Meta` class describes additional properties about the particular
`ModelForm` class it belongs to. Each `Meta` class must at a bare
minimum supply a `model` field, which references back to the model the
`ModelForm` inheriting class should relate to. Our `UserForm` class is
therefore associated with the `User` model, for example. As of Django
1.7, you also need to specify, `fields` or `exclude` to indicate which
fields associated with the model should be present on the form.

Here we only want to show the fields, `username`, `email` and
`password`, associated with the `User` model, and the `website` and
`picture` associated with the `UserProfile` model. For the `user` field
within `UserProfile` we will need to make this association when we
register the user.

You'll also notice that `UserForm` includes a definition of the
`password` attribute. While a `User` model instance contains a
`password` attribute by default, the rendered HTML form element will not
hide the password. If a user types a password, the password will be
visible. By updating the `password` attribute, we can then specify that
the `CharField` instance should hide a user's input from prying eyes
through use of the `PasswordInput()` widget.

Finally, remember to include the required classes at the top of the
`forms.py` module!

{lang="python",linenos=off}
	from django import forms
	from django.contrib.auth.models import User
	from rango.models import Category, Page, UserProfile

### Creating the `register()` View

Next we need to handle both the rendering of the form, and the
processing of form input data. Within Rango's `views.py` file, add the
following view function:

{lang="python",linenos=off}
	from rango.forms import UserForm, UserProfileForm

	...

	def register(request):
		# A boolean value for telling the template whether the registration was successful.
		# Set to False initially. Code changes value to True when registration succeeds.
		registered = False

		# If it's a HTTP POST, we're interested in processing form data.
		if request.method == 'POST':
			# Attempt to grab information from the raw form information.	
			# Note that we make use of both UserForm and UserProfileForm.
			 user_form = UserForm(data=request.POST)
			 profile_form = UserProfileForm(data=request.POST)

		# If the two forms are valid...
		if user_form.is_valid() and profile_form.is_valid():
			# Save the user's form data to the database.
			user = user_form.save()
			
			# Now we hash the password with the set_password method.
			# Once hashed, we can update the user object.
			user.set_password(user.password)
			user.save()

			# Now sort out the UserProfile instance.
			# Since we need to set the user attribute ourselves, we set commit=False.
			# This delays saving the model until we're ready to avoid integrity problems.
			profile = profile_form.save(commit=False)
			profile.user = user
			
			# Did the user provide a profile picture?
			# If so, we need to get it from the input form and 
			#put it in the UserProfile model.
			if 'picture' in request.FILES:
				profile.picture = request.FILES['picture']
			
			# Now we save the UserProfile model instance.
			profile.save()
			
			# Update our variable to tell the template registration was successful.
			registered = True
		
		# Invalid form or forms - mistakes or something else?
		# Print problems to the terminal.
		# They'll also be shown to the user.
		else:
			print user_form.errors, profile_form.errors

	# Not a HTTP POST, so we render our form using two ModelForm instances.
	# These forms will be blank, ready for user input.
	else:
		user_form = UserForm()
		profile_form = UserProfileForm()

	# Render the template depending on the context.
	return render(request,
			'rango/register.html',
			{'user_form': user_form, 'profile_form': profile_form, 
				'registered': registered} )


While the view looks pretty complicated it is very similar in nature to how we implemented the add category and add page views. However, we had to also handle two distinct `ModelForm` instances - one for
the `User` model, and one for the `UserProfile` model. We also need to
handle a user's profile image, if he or she chooses to upload one.

We also establish a link between the two model instances that we create.
After creating a new `User` model instance, we reference it in the
`UserProfile` instance with the line `profile.user = user`. This is
where we populate the `user` attribute of the `UserProfileForm` form,
which we hid from users in Section login-formclasses-label.

### Creating the *Registration* Template

Now create a new template file, `rango/register.html` and add the
following code:

``` {.sourceCode .html}
<!DOCTYPE html>
<html>
    <head>
        <title>Rango</title>
    </head>

    <body>
        <h1>Register with Rango</h1>

        {% if registered %}
        Rango says: <strong>thank you for registering!</strong>
        <a href="/rango/">Return to the homepage.</a><br />
        {% else %}
        Rango says: <strong>register here!</strong><br />

        <form id="user_form" method="post" action="/rango/register/"
                enctype="multipart/form-data">

            {% csrf_token %}

            <!-- Display each form. The as_p method wraps each element in a paragraph
                 (<p>) element. This ensures each element appears on a new line,
                 making everything look neater. -->
            {{ user_form.as_p }}
            {{ profile_form.as_p }}

            <!-- Provide a button to click to submit the form. -->
            <input type="submit" name="submit" value="Register" />
        </form>
        {% endif %}
    </body>
</html>
```

This HTML template makes use of the `registered` variable we used in our
view indicating whether registration was successful or not. Note that
`registered` must be `False` in order for the template to display the
registration form - otherwise, apart from the title, only a success
message is displayed.

> **warning**
>
> You should be aware of the `enctype` attribute for the `<form>`
> element. When you want users to upload files from a form, it's an
> absolute *must* to set `enctype` to `multipart/form-data`. This
> attribute and value combination instructs your browser to send form
> data in a special way back to the server. Essentially, the data
> representing your file is split into a series of chunks and sent. For
> more information, check out [this great Stack Overflow
> answer](http://stackoverflow.com/a/4526286). You should also should
> remember to include the CSRF token, too. Ensure that you include
> `{% csrf_token %}` within your `<form>` element.

### The `register()` View URL Mapping

Now we can add a URL mapping to our new view. In `rango/urls.py` modify
the `urlpatterns` tuple as shown below:

``` {.sourceCode .python}
urlpatterns = patterns('',
    url(r'^$', views.index, name='index'),
    url(r'^about/$', views.about, name='about'),
    url(r'^category/(?P<category_name_slug>\w+)$', views.category, name='category'),
    url(r'^add_category/$', views.add_category, name='add_category'),
    url(r'^category/(?P<category_name_slug>\w+)/add_page/$', views.add_page, name='add_page'),
    url(r'^register/$', views.register, name='register'), # ADD NEW PATTERN!
    )
```

The newly added pattern points the URL `/rango/register/` to the
`register()` view.

### Linking Together

Finally, we can add a link pointing to that URL in our homepage
`index.html` template. Underneath the link to the category addition
page, add the following hyperlink.

``` {.sourceCode .html}
<a href="/rango/register/">Register Here</a>
```

### Demo

Easy! Now you'll have a new hyperlink with the text `Register Here`
that'll take you to the registration page. Try it out now! Start your
Django development server and try to register a new user account. Upload
a profile image if you wish. Your registration form should look like the
one illustrated in Figure fig-rango-register-form.

![A screenshot illustrating the basic registration form you create as
part of this tutorial.](../images/rango-register-form.png)

Upon seeing the message indicating your details were successfully
registered, the database should have two new entries in its tables
corresponding to the `User` and `UserProfile` models.

Adding Login Functionality
--------------------------

With the ability to register accounts completed, we now need to add
login in functionality. To achieve this we will need to undertake the
workflow below:

-   Create a login in view to handle user credentials
-   Create a login template to display the login form
-   Map the login view to a url
-   Provide a link to login from the index page

### Creating the `login()` View

In `rango/views.py` create a new function called `user_login()` and add
the following code:

``` {.sourceCode .python}
def user_login(request):

    # If the request is a HTTP POST, try to pull out the relevant information.
    if request.method == 'POST':
        # Gather the username and password provided by the user.
        # This information is obtained from the login form.
        # We use request.POST.get('<variable>') as opposed to request.POST['<variable>'],
        # because the request.POST.get('<variable>') returns None, if the value does not exist,
        # while the request.POST['<variable>'] will raise key error exception
        username = request.POST.get('username')
        password = request.POST.get('password')

        # Use Django's machinery to attempt to see if the username/password
        # combination is valid - a User object is returned if it is.
        user = authenticate(username=username, password=password)

        # If we have a User object, the details are correct.
        # If None (Python's way of representing the absence of a value), no user
        # with matching credentials was found.
        if user:
            # Is the account active? It could have been disabled.
            if user.is_active:
                # If the account is valid and active, we can log the user in.
                # We'll send the user back to the homepage.
                login(request, user)
                return HttpResponseRedirect('/rango/')
            else:
                # An inactive account was used - no logging in!
                return HttpResponse("Your Rango account is disabled.")
        else:
            # Bad login details were provided. So we can't log the user in.
            print "Invalid login details: {0}, {1}".format(username, password)
            return HttpResponse("Invalid login details supplied.")

    # The request is not a HTTP POST, so display the login form.
    # This scenario would most likely be a HTTP GET.
    else:
        # No context variables to pass to the template system, hence the
        # blank dictionary object...
        return render(request, 'rango/login.html', {})
```

This view may seem rather complicated as it has to handle a variety of
situations. Like in previous examples, the `user_login()` view handles
form rendering and processing.

First, if the view is accessed via the HTTP GET method, then the login
form is displayed. However, if the form has been posted via the HTTP
POST method, then we can handle processing the form.

If a valid form is sent, the username and password are extracted from
the form. These details are then used to attempt to authenticate the
user (with Django's `authenticate()` function). `authenticate()` then
returns a `User` object if the username/password combination exists
within the database - or `None` if no match was found.

If we retrieve a `User` object, we can then check if the account is
active or inactive - and return the appropriate response to the client's
browser.

However, if an invalid form is sent, because the user did not add both a
username and password the login form is presented back to the user with
form error messages (i.e. username/password is missing).

Of particular interest in the code sample above is the use of the
built-in Django machinery to help with the authentication process. Note
the use of the `authenticate()` function to check whether the username
and password provided match to a valid user account, and the `login()`
function to signify to Django that the user is to be logged in.

You'll also notice that we make use of a new class,
`HttpResponseRedirect`. As the name may suggest to you, the response
generated by an instance of the `HttpResponseRedirect` class tells the
client's browser to redirect to the URL you provide as the argument.
Note that this will return a HTTP status code of 302, which denotes a
redirect, as opposed to an status code of 200 i.e. OK. See the [official
Django documentation on
Redirection](https://docs.djangoproject.com/en/1.7/ref/request-response/#django.http.HttpResponseRedirect),
to learn more.

All of these functions and classes are provided by Django, and as such
you'll need to import them, so add the following imports to
`rango/views.py`:

``` {.sourceCode .python}
from django.contrib.auth import authenticate, login
from django.http import HttpResponseRedirect, HttpResponse
```

### Creating a *Login* Template

With our new view created, we'll need to create a new template allowing
users to login. While we know that the template will live in the
`templates/rango/` directory, we'll leave you to figure out the name of
the file. Look at the code example above to work out the name. In your
new template file, add the following code:

``` {.sourceCode .html}
<!DOCTYPE html>
<html>
    <head>
        <!-- Is anyone getting tired of repeatedly entering the header over and over?? -->
        <title>Rango</title>
    </head>

    <body>
        <h1>Login to Rango</h1>

        <form id="login_form" method="post" action="/rango/login/">
            {% csrf_token %}
            Username: <input type="text" name="username" value="" size="50" />
            <br />
            Password: <input type="password" name="password" value="" size="50" />
            <br />

            <input type="submit" value="submit" />
        </form>

    </body>
</html>
```

Ensure that you match up the input `name` attributes to those that you
specified in the `user_login()` view - i.e. `username` for the username,
and `password` for password. Don't forget the `{% csrf_token %}`,
either!

### Mapping the Login View to a URL

With your login template created, we can now match up the `user_login()`
view to a URL. Modify Rango's `urls.py` file so that its `urlpatterns`
tuple now looks like the code below:

``` {.sourceCode .python}
urlpatterns = patterns('',
    url(r'^$', views.index, name='index'),
    url(r'^about/$', views.about, name='about'),
    url(r'^category/(?P<category_name_slug>\w+)$', views.category, name='category'),
    url(r'^add_category/$', views.add_category, name='add_category'),
    url(r'^category/(?P<category_name_slug>\w+)/add_page/$', views.add_page, name='add_page'),
    url(r'^register/$', views.register, name='register'),
    url(r'^login/$', views.user_login, name='login'),
    )
```

### Linking Together

Our final step is to provide users of Rango with a handy link to access
the login page. To do this, we'll edit the `index.html` template inside
of the `templates/rango/` directory. Find the previously created
category addition and registration links, and add the following
hyperlink underneath. You may wish to include a line break (`<br />`)
before the link.

``` {.sourceCode .python}
<a href="/rango/login/">Login</a>
```

If you like, you can also modify the header of the index page to provide
a personalised message if a user is logged in, and a more generic
message if the user isn't. Within the `index.html` template, find the
header, as shown in the code snippet below.

``` {.sourceCode .python}
<h1>Rango says..hello world!</h1>
```

Replace this header with the following markup and Django template code.
Note that we make use of the `user` object, which is available to
Django's template system via the context. We can tell from this object
if the user is logged in (authenticated). If he or she is logged in, we
can also obtain details about him or her.

``` {.sourceCode .python}
{% if user.is_authenticated %}
<h1>Rango says... hello {{ user.username }}!</h1>
{% else %}
<h1>Rango says... hello world!</h1>
{% endif %}
```

As you can see we have used Django's Template Language to check if the
user is authenticated with `{% if user.is_authenticated %}`. The context
variable which we pass through to the template will include a user
variable if the user is logged in - so we can check whether they are
authenticated or not. If so they will receive a personalised greeting in
the header, i.e. `Rango says... hello leifos!`. Otherwise, the generic
`Rango says... hello world!` header is displayed.

### Demo

Check out Figure fig-rango-login-message for screenshots of what
everything should look like.

![Screenshots illustrating the header users receive when not logged in,
and logged in with username
`somebody`.](../images/rango-login-message.png)

With this completed, user logins should now be completed! To test
everything out, try starting Django's development server and attempt to
register a new account. After successful registration, you should then
be able to login with the details you just provided.

Restricting Access
------------------

Now that users can login to Rango, we can now go about restricting
access to particular parts of the application as per the specification,
i.e. that only registered users can add categories and pages. With
Django, there are two ways in which we can achieve this goal:

-   directly, by examining the `request` object and check if the user is
    authenticated, or,
-   using a convenience *decorator* function that check if the user is
    authenticated.

The direct approach checks to see whether a user is logged in, via the
`user.is_authenticated()` method. The `user` object is available via the
`request` object passed into a view. The following example demonstrates
this approach.

``` {.sourceCode .python}
def some_view(request):
    if not request.user.is_authenticated():
        return HttpResponse("You are logged in.")
    else:
        return HttpResponse("You are not logged in.")
```

The second approach uses [Python
decorators](http://wiki.python.org/moin/PythonDecorators). Decorators
are named after a [software design pattern by the same
name](http://en.wikipedia.org/wiki/Decorator_pattern). They can
dynamically alter the functionality of a function, method or class
without having to directly edit the source code of the given function,
method or class.

Django provides a decorator called `login_required()`, which we can
attach to any view where we require the user to be logged in. If a user
is not logged in and they try to access a page which calls that view,
then the user is redirected to another page which you can set, typically
the login page.

### Restricting Access with a Decorator

To try this out, create a view in Rango's `views.py` file, called
`restricted()` and add the following code:

``` {.sourceCode .python}
@login_required
def restricted(request):
    return HttpResponse("Since you're logged in, you can see this text!")
```

Note that to use a decorator, you place it *directly above* the function
signature, and put a `@` before naming the decorator. Python will
execute the decorator before executing the code of your function/method.
To use the decorator you will have to import it, so also add the
following import:

``` {.sourceCode .python}
from django.contrib.auth.decorators import login_required
```

We'll also add in another pattern to Rango's `urlpatterns` tuple in the
`urls.py` file. Our tuple should then look something like the following
example. Note the inclusion of mapping of the `views.restricted` view -
this is the mapping you need to add.

``` {.sourceCode .python}
urlpatterns = patterns('',
    url(r'^$', views.index, name='index'),
    url(r'^add_category/$', views.add_category, name='add_category'),
    url(r'^register/$', views.register, name='register'),
    url(r'^login/$', views.user_login, name='login'),
    url(r'^(?P<category_name_slug>\w+)', views.category, name='category'),
    url(r'^restricted/', views.restricted, name='restricted'),
    )
```

We'll also need to handle the scenario where a user attempts to access
the `restricted()` view, but is not logged in. What do we do with the
user? The simplest approach is to redirect his or her browser. Django
allows us to specify this in our project's `settings.py` file, located
in the project configuration directory. In `settings.py`, define the
variable `LOGIN_URL` with the URL you'd like to redirect users to that
aren't logged in, i.e. the login page located at `/rango/login/`:

``` {.sourceCode .python}
LOGIN_URL = '/rango/login/'
```

This ensures that the `login_required()` decorator will redirect any
user not logged in to the URL `/rango/login/`.

Logging Out
-----------

To enable users to log out gracefully it would be nice to provide a
logout option to users. Django comes with a handy `logout()` function
that takes care of ensuring that the user is logged out, that their
session is ended, and that if they subsequently try to access a view, it
will deny them access.

To provide log out functionality in `rango/views.py` add the a view
called `user_logout()` with the following code:

``` {.sourceCode .python}
from django.contrib.auth import logout

# Use the login_required() decorator to ensure only those logged in can access the view.
@login_required
def user_logout(request):
    # Since we know the user is logged in, we can now just log them out.
    logout(request)

    # Take the user back to the homepage.
    return HttpResponseRedirect('/rango/')
```

With the view created, map the URL `/rango/logout/` to the
`user_logout()` view by modifying the `urlpatterns` tuple in Rango's
`urls.py` module:

``` {.sourceCode .python}
urlpatterns = patterns('',
    url(r'^$', views.index, name='index'),
    url(r'^about/$', views.about, name='about'),
    url(r'^category/(?P<category_name_slug>\w+)$', views.category, name='category'),
    url(r'^add_category/$', views.add_category, name='add_category'),
    url(r'^category/(?P<category_name_slug>\w+)/add_page/$', views.add_page, name='add_page'),
    url(r'^register/$', views.register, name='register'),
    url(r'^login/$', views.user_login, name='login'),
    url(r'^restricted/$', views.restricted, name='restricted'),
    url(r'^logout/$', views.user_logout, name='logout'),
    )
```

Now that all the machinery for logging a user out has been completed,
it'd be handy to provide a link from the homepage to allow users to
simply click a link to logout. However, let's be smart about this: is
there any point providing the logout link to a user who isn't logged in?
Perhaps not - it may be more beneficial for a user who isn't logged in
to be given the chance to register, for example.

Like in the previous section, we'll be modifying Rango's `index.html`
template, and making use of the `user` object in the template's context
to determine what links we want to show. Find your growing list of links
at the bottom of the page and replace it with the following HTML markup
and Django template code. Note we also add a link to our restricted page
at `/rango/restricted/`.

``` {.sourceCode .html}
{% if user.is_authenticated %}
<a href="/rango/restricted/">Restricted Page</a><br />
<a href="/rango/logout/">Logout</a><br />
{% else %}
<a href="/rango/register/">Register Here</a><br />
<a href="/rango/login/">Login</a><br />
{% endif %}

<a href="/rango/about/">About</a><br/>
<a href="/rango/add_category/">Add a New Category</a><br />
```

Simple - when a user is authenticated and logged in, he or she can see
the `Restricted Page` and `Logout` links. If he or she isn't logged in,
`Register Here` and `Login` are presented. As `About` and
`Add a New Category` are not within the template conditional blocks,
these links are available to both anonymous and logged in users.

Exercises
---------

This chapter has covered several important aspects of managing user
authentication within Django. We've covered the basics of installing
Django's `django.contrib.auth` application into our project.
Additionally, we have also shown how to implement a user profile model
that can provide additional fields to the base
`django.contrib.auth.models.User` model. We have also detailed how to
setup the functionality to allow user registrations, login, logout, and
to control access. For more information about user authentication and
registration consult [Django's official documentation on
Authentication](https://docs.djangoproject.com/en/1.7/topics/auth/).

-   Customise the application so that only registered users can
    add/edit, while non-registered can only view/use the
    categories/pages. You'll also have ensure links to add/edit pages
    appear only if the user browsing the website is logged in.
-   Provide informative error messages when users incorrectly enter
    their username or password.

In most applications you are going to require different levels of
security when registering and managing users - for example, making sure
the user enters an email address that they have access to, or sending
users passwords that they have forgotten. While we could extend the
current approach and build all the necessary infrastructure to support
such functionality a `django-registration-redux` application has been
developed which greatly simplifies the process - visit
<https://django-registration-redux.readthedocs.org> to find out more
about using this package. Templates can be found at:
<https://github.com/macdhuibh/django-registration-templates>