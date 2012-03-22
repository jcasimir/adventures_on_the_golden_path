# Adventures on the Golden Path

### Opening

Do you remember 2005? I do. I remember sitting in a chair eating my lunch at the Electronic Frontier Foundation. I was there working for the summer, creating curriculum materials around EFF issues. I was teaching high school computer science at the time, and it was important to me that my students learn about the politics and ideas in our culture.

And I was building a web app in JSP. I hated JSP. Over lunch I watched this video of a kid with a funny accent saying "Whoops!" while building a web application in just a few minutes.

My life changed watching that video. That afternoon, I deleted my JSP project and started over with Ruby on Rails. That was the last day I ever wrote Java, a skill I'd been practicing for six years.

### Love

Some of you were in the community then. Why did we fall in love?

Rails took this obscure little Japanese language and gave us the tools to do awesome things. But it wasn't just the ability to do great things -- Rails didn't offer any really new abilities over Java and ASP.

It was about speed and happiness. As a developer, shipping software is fun -- so speed and happiness are usually the same thing. We could build applications quickly, maybe even 10X quicker.

Back then, just before the first the Pragmatic Programmers invented the idea of a "beta book", the community was just a few hundred people trying to figure out the potential.

### Fast-Forward

Now we're seven years in. There are probably a hundred thousand developers who have, at one point, built a Rails application.

We can no longer claim a 10X speed advantage. Not because we've slowed down, but because many of the ideas which were the key insights of Ruby and Rails have become widespread. 

And that's a good thing. This is not a zero-sum game. Ideas we incubated, argued, and proved are now making the world a beter place. That's really cool.

But what about us? I think we're letting things slide. There are rough spots in our practice that we can both improve and, more importantly steer the community towards.

### Steering

I'm a developer, I do what I want!

Rails has always been about telling you what to do. This is how your project files & folders should be structured. This is how you do MVC. This is how you design database tables. Rails even tells us when to "person" becomes "people". It's amazing that we can get anything done.

But, of course, those rules and strong suggestions are what allows things to be so fast. I can grab just about any Rails application and, in minutes, understand what it's doing. Having conventions in our application code allows us tremendous reuse across our own projects and across the community in gems and services. Embrace the constraints.

### Steering, Better

The golden path, as it's known, has some bumps. Some of these have been around since 2005. Let's discuss a few ways that we tweak what we do to promote consistency, get closer to object-oriented ideals, while making developing applications a more joyful experience.

### This Ain't No Router Rewrite

The first component of your application encountered when responding to a request is the router. The router, at this point, has been rewritten about 82 times. We have made a lot of progress here.

When you create a new application, the routes.rb has *52* lines of comments. Attempting to embed a psuedo-exhaustive list of the API features is unnecessary and detracts from the golden path.

Here's what those comments should show:
	* resource :article
	* root :to => "articles#index"
	* Learn more: http://guides.rubyonrails.org/routing.html

Done.

### But Maybe It Is

I generally like the syntax for the routes file itself. But I don't like explaining the routing helpers. Here's what I tell people:

* run `rake routes`
* find the name in the left column of the route you want
* if there isn't a name there, imagine them trickling down
* look at the path to see if a parameter is required
* append `"_path"` most of the time to get a relative path
* append `"_url"` most of the time to get an absolute URL

It only takes them about 200 times of looking at this table and writing paths for it to make sense.

When we write paths like `customer_path(5)` or `edit_customer_path`, why is `customer` in the name? To provide a namespace -- that's kinda awkward.

We could, maybe, add a `path` method to the `Customer` class but that'd be mixing routing concerns into the model which is no good. What if, instead, we had an easily accessible application object which could be responsible for it. And what if we took the pattern from ARel to build up paths lazily?

Could we do this:
	* `index` / `create` with `Routes.customers`
	* `show` / `update` / `destroy` with `Routes.customer(5)`
	* `edit` with `Routes.customer(5).edit`
	
<< Proof of concept? >>

### At the System Level

Another question I hate hearing is "From the terminal, why do we sometimes use `rake` and sometimes we use `rails`?"

Well, it's ummm, well, that's how we've always done it.

But why?

We create apps with `rails new appname`. From there, let's never say `rails` again.

* `rake generate model Article`
* `rake server`
* `rake console`

Is it tricky to have Rake handle command line arguments this way? Yes. But is it possible? Or could Rake be changed? Of course.

https://gist.github.com/2158793

## Models

Let's drop down to the model layer. 

### Model Security

Let's start with a history lesson that will seem totally unrelated.

#### Seemingly Unrelated History

What was the most painful part of upgrading to Rails 3? For me, it was dealing with `html_safe`. Without causing an exception, I had strings all over my apps which were outputting escaped HTML. Tracking them down was difficult and time consuming.

But it was worth the pain, right? During the Rails 2 lifecycle it became clear that developers were forgetting to use the `h` helper to escape untrusted strings. Many apps were thus vulnerable to script injection attacks.

So we all bit the bullet and flipped the default. Now strings are, by default, untrused. We had to do some annoying, boring work to make our old apps work.

`html_safe` still trips me up, frequently, but the pain is worth it. I know that, unless I intentionally open holes, I'll be safe from script injection.

#### Mass-Assignment Security

Programmers cannot be trusted to remember things. It has to be thrown in our faces. The recent Github vulnerability demonstrates that even one of the best teams on earth forgets.

What happens when you generate a model in Rails?

```
jsblogger_advanced$ rails generate model Moderator first_name:string last_name:string account_level:integer
      invoke  active_record
      create    db/migrate/20120322151240_create_moderators.rb
      create    app/models/moderator.rb
      invoke    rspec
      create      spec/models/moderator_spec.rb
jsblogger_advanced$ cat app/models/moderator.rb 
class Moderator < ActiveRecord::Base
end
```

When you create a new app, the `routes.rb` has over 50 lines of tips for what you might want to do. But a model, arguably the most important component in the system has zero.

Here's what should happen:

```
class Moderator < ActiveRecord::Base
  # Create a whitelist of attributes to allow mass-assignment.
  # Remove any that you don't want easily modified from the outside
  attr_accessible :first_name, :last_name, :account_level
end
```

Then you start your console, and everything is cool:

```
jsblogger_advanced$ rails console
Loading development environment (Rails 3.2.1)
1.9.2-p318 :001 > Moderator.new
 => #<Moderator id: nil, first_name: nil, last_name: nil, account_level: nil, created_at: nil, updated_at: nil> 
1.9.2-p318 :002 > 
```

Unless you have models which don't have `attr_accessible` or `attr_protected`:

```
jsblogger_advanced$ rails console
Loading development environment (Rails 3.2.1)
WARNING: ActiveRecord::Base models [Article, Comment] have neither attr_accessible nor attr_protected
1.9.2-p318 :001 > Moderator.new
 => #<Moderator id: nil, first_name: nil, last_name: nil, account_level: nil, created_at: nil, updated_at: nil> 
1.9.2-p318 :002 > 
```

Which is a nice reminder that you need to be more careful about security. Unless, that is, you set this option in your environment:

```
config.raise_security_errors = true
```

Then when you try to start the console/server:

```
jsblogger_advanced$ rails console
Loading development environment (Rails 3.2.1)
SECURITY ERROR DETECTED: ActiveRecord::Base models [Article, Comment] have neither attr_accessible nor attr_protected
Exiting...
jsblogger_advanced$ 
```

This should be enabled by default in test and production. There are other good ideas coming from the Github vulerability, but this one comes at almost no cost.

#### Persistence and Business Logic

If you've been anywhere near an "OO in Rails" conversation over the past year, you've heard griping about how our models mix persistence and business logic.

We know from SOLID that a class having two responsibilities leads to a bad situation. There's nothing stopping you from separating them right now.

But that's not what the Rails Way does. Rails, like a watchful mother, should help you make the right choices:

```
jsblogger_advanced$ rails generate model Moderator first_name:string last_name:string
      invoke  active_record
      create    db/migrate/20120322163838_create_moderators.rb
      create    app/models/moderator.rb
      create    app/data/moderator_data.rb
      invoke    rspec
      create      spec/models/moderator_spec.rb
      create      spec/data/moderator_data_spec.rb      
```

See what I did there?

```
jsblogger_advanced$ cat app/models/moderator.rb 
class Moderator
	extend ModeratorData
end
jsblogger_advanced$ cat app/data/moderator_data.rb
module ModeratorData
	extend ActiveRecord::Record
end 
```

Aaron Patterson has already worked on bringing in ActiveRecord as a module rather than through inheritance. Let's go ahead and rename it to `ActiveRecord::Record` since there's no domain concept of a `"Base"`.

Down the line, our model might just include things like this:

```
jsblogger_advanced$ cat app/models/moderator.rb 
class Moderator
	extend ModeratorData

	def last_active?
		posts.most_recent.created_at
	end
end
```

Where all the data connections and data integrity are handled over in the Module:

```
jsblogger_advanced$ cat app/data/moderator_data.rb
module ModeratorData
	extend ActiveRecord::Record

	has_many   :posts
	belongs_to :blog

	validates_presence_of :first_name, :last_name
end 
```

Rails can show us the right way to separate concerns.

### Controllers & Views

I'm tempted to talk about composition over inheritance and how `ApplicationController` should be a module. But let's skip to the real issue.

#### Instance Variables

Instance variables, people. I think instance variables are a code smell in Ruby. That means your current controllers and views STINK!

Do you know how the instance variables from inside the controller instance get over to the view template? It ain't pretty! Worse, it obliterates object encapsulation.

Surely the solution would take masterminds of programming many months of intense hacking? No.

```
jsblogger_advanced$ rails generate scaffold Message subject:string body:text
      invoke  active_record
      #...
      invoke  scaffold_controller
      create    app/controllers/messages_controller.rb
      invoke    erb
      create      app/views/messages
      create      app/views/messages/index.html.erb
      #...
```

So far nothing of interest...

```
jsblogger_advanced$ cat app/controllers/messages_controller.rb 
class MessagesController < ApplicationController
	attr_accessor :messages, :message
	helper_method :messages
	helper_method :message

  # GET /messages
  # GET /messages.json
  def index
    self.messages = Message.all

    respond_to do |format|
      format.html # index.html.erb
      format.json { render json: @messages }
    end
  end
  #...
end
```

And in the view...

```
jsblogger_advanced$ cat app/views/messages/index.html.erb 
<h1>Listing messages</h1>

<!-- ... -->

<% messages.each do |message| %>
  <tr>
    <td><%= message.subject %></td>
    <td><%= message.body %></td>
    <!-- ... -->
  </tr>
<% end %>

<!-- ... -->
```

No more instance variables in your views.

Want to clean up the syntax a bit? Let's go all-in:

```
module AttributeViewer
  def attr_viewable(*names)
    names.each do |name|
      attr_accessor name
      helper_method name
    end
  end
end

class ApplicationController < ActionController::Base
  extend AttributeViewer
end

class MessagesController < ApplicationController
  attr_viewable :message, :messages

  # GET /messages
  # GET /messages.json
  def index
    self.messages = Message.all
  end
end
```

Now you can almost pretend that Views and Controllers aren't totally intertwined...



### Conclusion Ideas

* System
	* Rake instead of Rails

* Routes
  * Strip the comments to just the best practices
  * Just treat the Router as an object
    Route.article(id), Route.articles, Route.edit_article
  * What if Routes worked like ARel:
    Route.article(id).edit
    Route.article(id).destroy

* Controllers/Views
  * Instance Variables vs Accessors / "Helper Methods"

* Models
  * attr_accessible in comments
  * attr_accessible by default
  * ActiveRecord as a module
    * Make private?
  * Persistence folder
  