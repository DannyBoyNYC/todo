https://frontendmasters.com/blog/vanilla-javascript-todomvc/

https://github.com/1Marc/modern-todomvc-vanillajs - added branches for TypeScript support, used lit-html for performant rendering, and an app-architecture branch for how to architect larger apps

Todo Project:
https://todomvc.com/

Things - https://culturedcode.com/things/

A tiny + on the desktop app and a blue (+) on the mobile apps. The emphasis is more on knowing that Command-N will do the trick. I do enjoy the little feature that you can drag the mobile (+) where you want it before adding a new item. Notably, you add a new item before you add any text or information to the item. Metadata is pretty limited, preferring that you group things in various ways (areas, projects, headings), although there is things like tags if you want them. If any metadata is emphasized, it is due dates.

Todoist - https://todoist.com/

web-based, More metadata is exposed here. Tasks have a description which you automatically some of is interesting. Plus tags, plus due dates, plus collapsible groups.

Notice that adding a new task is also relatively chill. Unlike Things, pressing the + Add task button doesn’t automatically create a new task, but instead expands into a fleshed out task adder:

The task name is the only required thing, which makes sense. It would be obnoxious if anything else was. It feels like a reasonable choice to allow you to add all this metadata immediately, but certainly not the only path. They could do it such that you add the task first, then optionally add more to it if desired as an optional next step if you wanted.

TeuxDeux - https://teuxdeux.com/home?signup=web

TeuxDeux is big on having you put to-do items on individual days, although you can customize it if you want to. Having this kind of initially opinionated structure will have a huge influence on how people think about your app.

Notice there is almost no UI for adding new items. The way it works is that the next item in each column is automatically a live input, waiting for you get into it and add text. The extremely narrow item widths will also encourage you to get to the point darn quickly. You can see the markdown come through with the editing toolbar (Things also does Markdown, but only in the body of the description, which is hidden until opening an individual item.)

TodoMVC - https://todomvc.com/examples/vue/dist/#/

Note there is no submit/add button for new items, which weirds me out a little bit on the web. I’d say this then entirely relies on the submit even from the <form>, but in this case there is no <form> (?!), which then not only requires JavaScript but more JavaScript as they can’t just use that native event. It’s likely our app will require JavaScript as well, at least at first, as adding a backend language might be a trip too far for this series (but we’ll see, if y’all love it, lemme know).

There is no way we’re going to be as robust as some of these apps who have been working on their feature set for years and years. Let’s call it like this:

- Let’s emphasize the simple text input. Let’s see how easy we can make it to add a new item to one big single list.
- Let’s also emphasize the Add/+ button. Not only does it lean into HTML and accessibility correctness, but we can also make it part of our branding. Maybe we can borrow the Things approach of draggability someday.
- Just one big list for now, but let’s think extensibility when we store our data. It shouldn’t be a stretch to add multiple lists, tags, descriptions, or anything else.
- Add and Complete are the primary actions. For now, let’s borrow TodoMVC’s approach of double-click to edit unless we find something we like better while building.
- Let’s try and make drag-to-reorder happen.

Of course, we’d love to add user accounts, Markdown support, due dates, tagging, an API, bulk actions, recurring items, and who knows?? AI helpers? Integrations? If you absolutely knew that these features needed to be there for day one, then they should be part of the design process from day one. For us, not so much, so we’ll keep things simple and trim and build that firs

## Header

```html
<header>
  <svg ... />
  <h1>TODO</h1>
</header>
```

The `<h1>` is perhaps a little controversial. I often prefer “saving” that for whatever the main focus of the page is. But in this case since the name of the app is doing that for us, it kinda works. Otherwise a `<div>` or whatever is fine here.

## Form

```html
<form id="todo-form" class="todo-form">
  <label>
    <span class="screen-reader-text">New TODO</span>
    <input type="text" name="todo" />
  </label>
  <button>Add</button>
</form>
```

The `<form>` feels vitally important to me. Already in poking around at other web-based to-do apps I’ve seen some inaccessible forms which feels sad to me for something so simple.

The trick is:

- Actually using a <form>.
- Making sure the <input> is properly <label>ed.
- Offering a submit <button>.

I like having the id on the form for getting our hands on it in JavaScript like we’re likely going to want to. We can make the interactive event of adding a new to-do happen on the submit event. Plus that makes us ready for potentially adding more information into the form later if we wanted, like, say, a date picker for a deadline. The submit event will still work great for us, instead of something more specific like watching for the enter key being pressed while any specific input is focused.

I added the class for styling. That’s just a habit of mine, preferring to style with mostly classes for fairly flat specificity. If you have your own methods or use Tailwind or whatever, you likely already know what you’re doing and godspeed.

Notice my <label> doesn’t have a for attribute like you normally see, and that’s because the `<input>` is nested within it, which is a valid pattern. I like the pattern as it’s one less thing to screw up.

The actual text of the label we’re hiding, as-per the design. That’s also a little controversial, but I think is generally fine. It’s fairly clear what this form does, and if you were using a screen reader, the actual label text is still there even if we do hide it visually with CSS.

Last, the <button> is a lone wolf within the form, and that’ll make it be a submit button automatically. We could use an `<input type="submit" />` as well, which is fine, but I like buttons. You can put more stuff in them and their type is implied in simple situations like this.

## List

The interesting thing about the list is that it’s going to be dynamically generated. We’ll have as many to-do list items as our data tells us. We can start with just the wrapper:

```html
<ol id="todo-list" class="todo-list"></ol>
```

Again using the ol’ id/class one two punch there. I feel like an ordered list is appropriate somehow. Especially if we eventually add sorting of the list, then the order is very intentional and deserves that treatment. And the fact that we’re using a list at all feels right. Lists announce themselves as such in screen readers, including how many items are in the list, which may be a useful bit of information. That’s more than a bunch of `<div>`s would do!

The each list item becomes a `<li>`, but the extra stuff in there is going to beef it up. Let’s pause on the re-ordering and edit-ability for now and just focus on the item and the ability to complete it.

```html
<li id="something-unique">
  <button aria-label="Complete">
    <svg ... />
  </button>
  To-do text
</li>
```

I figure each list item will have a unique ID on it. Actions taken on it will need to be unique to that item so that’ll be the main identifier. Maybe we don’t need it yet but we probably will. No class name here as I feel like selecting .todo-list > li is probably fine. We could always add it.

Our design makes the interactive element to complete a to-do look like a checkbox, and we could use an `<input type="checkbox" />`, but I’m kinda torn. This is where visual design and interactive design collide. We didn’t think out the interaction of completing a to-do all the way. What happens when you click that thing? Does it instantly get marked as done? Then disappear? Maybe it instantly gets marked as done but doesn’t disappear from the list. Or maybe there is like a 5-second countdown before it gets removed, so you have a chance to undo it in case it was a mistake.

I kinda like the delay idea, which to me leads me more toward the path of using an `<input type="checkbox">` for that with the very obvious two states. We could really do it either way. If we keep the `<button>`, we’d just need to make sure to use an aria-pressed role for the two states accordingly.

Lastly, the position of that interactive element I have listed first here, before the text of the to-do, because that’s how it looks in the design. I could see the case being made though that in the HTML it should come after, just so that it reads nicer. Like as you tab through the list, you’d potentially hear the list item and then the button text offering to complete it. I’m not 100% sure what the right answer is there, so I think we’ll leave it first, because that’s the visual order, and I’ve heard it’s often best to match the source order and visual order.

## JavaScript

### Data

There are more ways and places to save data on the web than you can shake a stick at. Much like we slowed down at the beginning and did some design thinking at first, we might do well do do a little data thinking and consider the experience we’re trying to build.

For example, we might want to offer our TODO app for multiple users who log in to the website. If that’s the case, we’ll need to store the data in such a way that each user has their own set of to-dos associated with them. That would be fun! In that case we’d have to think about security and ensuring correct auth around data access. Maybe a bit much for a first crack though, so let’s table that for now.

Circling back to the front-end, it would be nice to get our data there ultimately as JSON. JSON is just so browser-friendly, as JavaScript can read it and loop over it and do stuff with it so easily. Not to mention built-in APIs.

That doesn’t mean we have to store data as JSON, but it’s not a terrible idea. We could use a more traditional database like MySQL, but then we’d need some kind of tool to help us return our data as JSON, like any decent ORM should be able to help with. SQLlite might be even better.

Or if we went with Postgres, it supports a better native json field type, and built-in utility functions like ROW_TO_JSON().

Still, just keeping the data as JSON sounds appealing. I’m not a huge data expert, but as I understand it there are JSON-first databases like MongoDB or CouchDB that might make perfect sense here. I’ve used Firebase a number of times before and their data storage is very JSON-like.

Let’s assume we’re going to use JSON data, and dunk that data as a string in localStorage. While this isn’t exactly a database, we can still nicely format our data, make it extensible, and pretend like we’re interfacing with a fancier database system.

What we’re giving up with localStorage is a user being able to access their data from anywhere. If they so much as open our TODO app in another browser, their data isn’t there. Let alone open the site on their phone or the like. Or do something blasphemous like clear their browser data. So it’s rudimentary, but it’s still “real” enough data storage for now.

### Data Shape

JSON can be an {} Object or [] Array. I’m thinking Array here, because arrays have this natural sense of order. We talked about being able to re-arrange to-do items at some point, and the position in the array could be all we need for that.

As best I know, as of ES2020, I believe Objects maintain order when you iterate over them in all the various ways. But I don’t think there is any reasonable way to change that order easily, without creating a brand new Object and re-inserting things in the newly desired order. Or, we’d have to keep an order value for each item, then update that as needed and always iterate based on that value. Sounds like too much work.

```js
[
  {
    /* to-do item */
  },
  {
    /* to-do item */
  },
];
```

(Astute readers will note that JSON doesn’t actually have comments, unless we fix it somehow.)

If we were supporting a multi-user system, perhaps all the data would be an object with each user ID being a key and the data being an array like above. One way or another, one use per Array.

Now what does each item look like? Based on our design, we really only need a few things:

Title (text of the to-do)
Complete (whether it is done or not)
We could decide that we don’t need “complete” because we’ll just delete it. But any to-do app worth it’s salt will be able to show you a list of completed items, and you can always delete them from there.

We talked about using the array order for the visual order of the to-dos. I think I’m fine with that for now, knowing we can always add ordering properties if we really feel like it. In fact, we can add whatever. We could add dates like a “created at”, “modified at”, “due date”, or “completed at” if we felt it would be useful. We could add tags. We could add a description. But these kind of things should be design and UX driven, as we well know by now. Don’t just go adding data speculatively, that tends to not end well.

When it comes to updating/deleting existing to-dos, we’re going to need a way to update just that item in the data. That’s why I brought up the array order again, because theoretically we could know which item we’re dealing with by the DOM order, then match that to the array order to find the right item. But something about that feels janky to me. I’d rather have a more solid-feeling one-to-one connection between the UI and the data. Maybe it’s just me, but it feels better. So let’s add a unique identifier to each todo.

So our now-list-of-three will look like this in the data:

```js
[
  {
    title: "Walk the dog", // string
    completed: false, // boolean
    id: "something-unique", // string
  },
  {
    // more!
  },
];
```

### Saving Data

There is one way to add a new item on our site: submitting the form. So if we get ahold of that form in JavaScript and watch for the submit event, we’re in business.

```js
const form = document.querySelector("#todo-form");

form.addEventListener("submit", (event) => {
  event.preventDefault();

  // Add to-do to data and render UI

  form.reset();
});
```
