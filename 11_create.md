# Creating an item

## Removing security
We are going to create a new item by submitting a HTTP form. We will build up the form from scratch. To keep things simple we will disable some of the security that Rails provides. Go to the `ApplicationCOntroller` and comment out `protect_from_forgery with: :exception`. We will re-introduce this later. 

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  # protect_from_forgery with: :exception
end
```

## The New Route
We can proceed by going to a URL that is typically used to allow a user to create a new resource or item.

```
http://localhost:3000/questions/new
```

This results in a conflicting error:

```
ActiveRecord::RecordNotFound in QuestionsController#show

	def show
		@question = Question.find(params[:id])
	end
```

Unfortunately the routing rule we have for `show` has captured this URL:

```ruby

# config/routes.rb
Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/:id' => 'questions#show', as: :question  
```

The show rule tells Rails to match any of the following URLs:
* questions/1
* questions/2
* questions/any
* questions/new

To proceed we need to define a new route and place it above the `question` route.

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new',  as: :new_question
  get 'questions/:id' => 'questions#show', as: :question
```

Note that Rails looks matching rules in order, from top down. If we did not place this new rule above `show`, then the `question` route would still match the URL and process the request. 

After applying this we end up with the next error:

```ruby
The action 'new' could not be found for QuestionsController
```

## New Action

We need to add a new action to the controller called `new`. This action needs to create a new Question that will be used by the `new` view .

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

	def index
		@questions = Question.all
	end

	def show
		question_id = params[:id]
		@question = Question.find(question_id)
	end

	def new
		@question = Question.new
	end

end
```

Note that this does not create a Question in the database, this is only created in memory.

Saving this and reloading the browse results in a missing template error:

```
Missing template questions/new
```

We can create the template with the following HTML:

```html
<!-- app/views/new.html.erb -->
<h3>New Question</h3>

<form action="/questions" method="post">
  <p>
    <label for="question_title">Title</label>
    <input type="text" name="title" id="question_title" />
  </p>
  <p>
    <label for="question_body">Body</label>
  </p>
  <p> 
    <textarea cols="30" rows="10" name="body" id="question_body"></textarea>
  </p>
  <p>
    <input type="submit" name="commit" value="Create Question" />
  </p>
</form>

```

The `action` attribute tells the browser which URL to use when the user clicks submit. When the user does click submit, the form data will be sent to the server. This form data will include the `name` attributes of the fields and the `value` or `text` content of the input fields. Rails will handle this request, and make these values available in the controller. 

## Create Route

If we now click `submit` the form will submit a post to '/questions'. This will result in yet another routing error. Let's fill out some data and see the error:

```
No route matches [POST] "/questions"
```

Our routes file contains the following route:
```
get 'questions' => 'questions#index'
```
That handles the same URL, however that is for a GET, we are doing `n HTTP POST. This means we need to aff another routing rule:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new', as: :new_question
  get 'questions/:id' => 'questions#show', as: :question

  post 'questions' => 'questions#create'

```

Attempting to submit the form again results in this error:

```
The action 'create' could not be found for QuestionsController
```

## Create Action

Next we can create that action:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question = Question.new
      question.title = params[:title]
      question.body = params[:body]
      question.save
      redirect_to questions_path
    end
end
```

As you can see Rails takes the form data and created a hash-like object called `params`. The key's in `params` match the the `name` of the input fields used in the HTML form.

The final line in the `create` action redirects the user back to the index page. In other words, after creating the item in the database, the server sends a redirect response to the users browser. The browser will then submit another request to the URL specified in the redirect response. This is done so that the user won't create multiple items after if they refresh the browser.


We can implement the view in a slightly different way to help with processing the data in the controller. Changing the `name` attributes in the view will simplify the controller. 


```html
<!-- app/views/new.html.erb -->
<h3>New Question</h3>

<form action="/questions" method="post">
  <p>
    <label for="question_title">Title</label>
    <input type="text" name="question[title]" id="question_title" />
  </p>
  <p>
    <label for="question_body">Body</label>
  </p>
  <p> 
    <textarea cols="30" rows="10" name="question[body]" id="question_body"></textarea>
  </p>
  <p>
    <input type="submit" name="commit" value="Create Question" />
  </p>
</form>

```

Now `params` will contain a key of `question`. The value of this item will be another 'hash-like' object which contains the key\value pairs needed for the body and title. This means we can change the controller to the following:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question = Question.new
      question_params = params[:question]
      question.title = question_params[:title]
      question.body = question_params[:body]
      question.save
      redirect_to questions_path
    end
end

```

As you may recall we can create an Active Record item by calling `create` and passing it a hash. If we apply this we can use the following code:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question_params = params[:question]
      Question.create(title: question_params[:title], body: question_params[:body])
      redirect_to questions_path
    end
end
```

Given that `question_params` is already a 'hash-like' object we could try to use that in the `create` method:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question_params = params[:question]
      Question.create(question_params)
      redirect_to questions_path
    end
end
``` 

Unfortunately this results in an error:

```
ActiveModel::ForbiddenAttributesError
```

This is because the `params` object is not a standard hash (thus neither is `question_params`). `params` is an object which helps to prevent us from update fields of our model which we did not intend to e.g. id. This is a security feature of Rails.

In order to create the model directly from the `question_params` we need to tell Rails to permit the fields we want to use when creating (or updating) a model. This can be done with the following code:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question_params = params[:question]
      question_params.permit!()
      Question.create(question_params)
      redirect_to questions_path
    end
end
```

A more conventional way of achieving this in Rails is by making sure the `:question` key is present in the `params` and then permitting the required fields. That is done like this:


```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
    def index
        @questions = Question.all
    end

    def show
        @question = Question.find(params[:id])
    end

    def new
      @question = Question.new
    end
    
    def create
      question_params = params.require(:question).permit(:title, :body)
      question = Question.create(question_params)
      redirect_to questions_path
    end
end
```

If the `params` object does not contain a `:question` key then Rails will throw a `ActionController::ParameterMissing ` exception.

## Completing the New View

At the moment the `new` view looks like this:

```html
<!-- app/views/new.html.erb -->
<h3>New Question</h3>

<form action="/questions" method="post">
  <p>
    <label for="question_title">Title</label>
    <input type="text" name="question[title]" id="question_title" />
  </p>
  <p>
    <label for="question_body">Body</label>
  </p>
  <p> 
    <textarea cols="30" rows="10" name="question[body]" id="question_body"></textarea>
  </p>
  <p>
    <input type="submit" name="commit" value="Create Question" />
  </p>
</form>
```

We can make use of the Rails helper methods to tidy this up:

```html
<!-- app/views/new.html.erb -->
<h3>New Question</h3>

<%= form_for(@question, url: questions_path) do |f| %>
    <p>
        <%= f.label :title %>
        <%= f.text_field :title %>
    </p>
    <p>
        <%= f.label :body %>
    </p>
    <p> 
        <%= f.text_area :body, cols: 30, rows: 10 %>
    </p>
    <p>
        <%= f.submit %>
    </p>
<% end %>
```

The `form_for` method is a powerful construct. Notice how the form object `f` is used in subsequent lines to build up the form.

If there was a `number` or a `price`, we could have used `f.number_field :price`

It is worth looking into the built in date selectors
* http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html#method-i-datetime_select
* http://apidock.com/rails/v4.2.1/ActionView/Helpers/DateHelper/date_select


More information on `form_for` can be found here:
* http://guides.rubyonrails.org/form_helpers.html#binding-a-form-to-an-object

`form_for` is actually smart enough to figure out the action URL based on the class of the model we give it. This means the following would still work:

```html
<!-- app/views/new.html.erb -->
<h3>New Question</h3>

<%= form_for(@question) do |f| %>
    <p>
        <%= f.label :title %>
        <%= f.text_field :title %>
    </p>
    <p>
        <%= f.label :body %>
    </p>
    <p> 
        <%= f.text_area :body, cols: 30, rows: 10 %>
    </p>
    <p>
        <%= f.submit %>
    </p>
<% end %>

```

## Restoring security
Now that we are using the `form_for` helper, we can re-instate the security restrictions that Rails provides. 


```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
end
```

## Exercise

Make it possible to add items to explore