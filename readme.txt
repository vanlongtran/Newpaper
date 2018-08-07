Ch 8 Lesson 
Django for Beginners by William S. Vincent

-Created a new Newspaper app with a custom user model using AbstractUser
 
-Always use a custom user model for all new Django projects because changing the default user model in the future is more compclicated than using a custom user model from the beginning.

-Wait until after we’ve created our new custom user model before running migrate to configure our database.

-superuser password is Password_01

-Updated UserCreationForm and UserChangeForm to use customUser model instead of default.
-Update admin.py to extend the existing UserAdmin class to use our new CustomUser model
# users/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .forms import CustomUserCreationForm, CustomUserChangeForm
from .models import CustomUser

class CustomUserAdmin(UserAdmin):
	add_form = CustomUserCreationForm
	form = CustomUserChangeForm
	list_display = ['email', 'username', 'age']
	model = CustomUser
admin.site.register(CustomUser, CustomUserAdmin)

Ch9
-Create custom login/logout/registration form with customUser in /templates/registration

Ch10
-Added bootstrap templaye to style site.
-Added 3rd party package django-crispy-forms for signup page

Ch11 
-Added password change.
-Added password reset. 
Can use Django's console backend setting to output email text for testing. Use a email service in live settings.
# newspaper_project/settings.py
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

Ch12
When using mail service update the following:
# newspaper_project/settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

# newspaper_project/settings.py
EMAIL_HOST = 'smtp.emailserviceprovider.net'
EMAIL_HOST_USER = 'id_username'
EMAIL_HOST_PASSWORD = 'id_password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True

Ch 13
-Created a dedicated articles app with CRUD functionality.

**I was able to finish chapter 14 and 15 lessons but then I went off and on my own and broken the app. After troubleshooting efforts, I resorted to rollbacking and redoing these two lessons.

Here are the git options for rollbacking
1) UNDO local file changes but NOT REMOVE your last commit
git reset --hard

2) UNDO local file changes AND REMOVE your last commit
git reset --hard HEAD^

3) KEEP local file changes and REMOVE ONLY your last commit
git reset --soft HEAD^

Ch14
set permissions on edit/delete so that only the author of an article can make such changes. Edit the default CreateView and set the author via the form_valid method.

Both ListView and DetailView use this TemplateResponseMixin to render the proper template so this mixin save from repeating code.

To restrict view access to only logged in users, Django has a LoginRequired mixin. Added this to the views to block anonymous users.
Add $login_url = 'login' to the views to redirect it to the login page

Ch15
Added comments in line with articles
Created Comment model that has a many to one foreign key relationship to Article.
Traditionally the name of the foreign key
field is simply the model it links to, so this field will be called article . The other two fields will be comment and author 
#Update articles/models.py and migrate
...
class Comment(models.Model):
	article = models.ForeignKey(Article, on_delete=models.CASCADE)
	comment = models.CharField(max_length=140)
	author = models.ForeignKey(
		settings.AUTH_USER_MODEL,
		on_delete=models.CASCADE,
	)
	def __str__(self):
		return self.comment
	def get_absolute_url(self):
		return reverse('article_list')

# articles/admin.py
from django.contrib import admin
from . import models
class CommentInline(admin.TabularInline):
	model = models.Comment
class ArticleAdmin(admin.ModelAdmin):
	inlines = [
		CommentInline,
	]
admin.site.register(models.Article, ArticleAdmin)
admin.site.register(models.Comment)

To display all comments related to a specific article. This is a “query” as we’re asking the database for a specific bit of information. Since we are working with a foreign key, we want to follow a relationship backward: for each Article look up related Comment models.

add a related_name attribute to our model which lets us explicitly set the name of this reverse relationship
# Update articles/models.py and migrate to update the db model
...
class Comment(models.Model):
	article = models.ForeignKey(
		Article,
		on_delete=models.CASCADE,
		related_name='comments' # new
	)

To add to view:
<div class="card-footer">
	{% for comment in article.comments.all %}
		<p>
			<span class="font-weight-bold">{{ comment.author }} &middot;</span>
			{{ comment }}
		</p>
	{% endfor %}
</div>
#To access each comment we’re calling article.comments.all which means first look at the article model, then comments which is the related name of the entire Comment model, and select all included.


For reference
https://github.com/wsvincent/djangoforbeginners/blob/master/ch15-comments/templates/base.html
