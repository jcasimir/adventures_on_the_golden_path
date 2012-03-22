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


### Conclusion Ideas

* System
	* Rake instead of Rails

* Models
	* attr_accessible in comments
	* attr_accessible by default
	* ActiveRecord as a module
		* Make private?
	* Persistence folder

* Controllers
	* Composition over Inheritance
	* Instance Variables vs Accessors / "Helper Methods"
	* Simplify 

* Helpers
	* Die
	* Something in SOLID about only providing what you need
	* Helpers as modules or classes

* Views
	* PJax?
	* Instance variables are dead
	* Draper, etc
		* Lessons learned?

* Routes
	* Strip the comments to just the best practices
	* Just treat the Router as an object
		Route.article(id), Route.articles, Route.edit_article
	* What if Routes worked like ARel:
		Route.article(id).edit
		Route.article(id).destroy

