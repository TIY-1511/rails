## Refactoring

We can clean up the code by removing some duplication:

Update the controller to look as follows

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  def index
    @questions = Question.all
  end

  def show
    @question = Question.find(params[:id])
  end

  def edit
    @question = Question.find(params[:id])
  end

  def update
    @question = Question.find(params[:id])
    @question.update(question_params)
    redirect_to question_path(@question)
  end

  def new
    @question = Question.new
  end

  def create
    question = Question.create(question_params)
    redirect_to question_path(question)
  end

  private 

  def question_params
    params.require(:question).permit(:title, :body)
  end

end

```

## Cleaning up the Routes

The routes.rb file is growing fairly rapidly considering that we only have one type of item in the system.

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new', as: :new_question
  get 'questions/:id' => 'questions#show', as: :question
  get 'questions/:id/edit' => 'questions#edit', as: :edit_question
  patch 'questions/:id' => 'questions#update'

  post 'questions' => 'questions#create'
```

We can see what routes we have by typing `rake routes` in the terminal:

```
$ rake routes
       Prefix Verb  URI Pattern                   Controller#Action
    questions GET   /questions(.:format)          questions#index
 new_question GET   /questions/new(.:format)      questions#new
     question GET   /questions/:id(.:format)      questions#show
edit_question GET   /questions/:id/edit(.:format) questions#edit
              PATCH /questions/:id(.:format)      questions#update
              POST  /questions(.:format)          questions#create
```

We have been following Rails conventions (which are basically REST conventions). This means that we can use a helper method in Rails to generate all of these route definitions and a bit more. Change routes.rb to the following:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  resources :questions
```

Now running rake routes produces the following:

```
$ rake routes
       Prefix Verb   URI Pattern                   Controller#Action
    questions GET    /questions(.:format)          questions#index
              POST   /questions(.:format)          questions#create
 new_question GET    /questions/new(.:format)      questions#new
edit_question GET    /questions/:id/edit(.:format) questions#edit
     question GET    /questions/:id(.:format)      questions#show
              PATCH  /questions/:id(.:format)      questions#update
              PUT    /questions/:id(.:format)      questions#update
              DELETE /questions/:id(.:format)      questions#destroy
```

All of the routes we have been using have been generated at runtime. As we have been following conventions, these generated ones match the routes we had manually entered - as well as the Routes helper methods. 

There are also a few more e.g. DELETE and PUT

PUT is synonymous with PATCH, it is there for backwards compatibility. DELETE is for deleting items. In a future session we will use the delete route. 


## Exercise

Refactor explore so that you are happy with the code...