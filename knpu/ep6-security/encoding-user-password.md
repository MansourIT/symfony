# Encoding User Password

Now of course our user system needs a real password. Having everyone type in "I like turtles" is not a great security system. Lets do that. First in our system, but this can be different in your system, we are going to store the passwords in the database. I mean you might do something different because if you have some sort of central log in system it might be responsible for check the password and then in your Symfony app you don't need to check the password. That's totally valid use case, but if you do need to check the password in your Symfony app then you're probably going to want to store it in your database.

In user, let's add a private password and make that a column. Now this will be the encoded password. You will obviously never store the plain text password. Now remember we left these three methods blank before in the user interface because you only need these if you actually have a password, so now that we have a get password method we will return this arrow password. Get salt can stay blank because we're going to use the bcrypt algorithm which has a built in that solves the mechanism. I'll come back to erase credentials in a second.

First, let's go do code generate, and add that setter for passwords. We can update it. The next step doctor migrations diff. We'll do bin console doctrine migrations migrate. Perfect. In order to set this password, we're going to need to take a plain text password and encode it through the bcrypted algorithm. There's a number of ways to do this, but here's the one that we're going to use. We're going to set the plain text password on our user object, and then via a doctrine listener, when the user object is saved, we will automatically read that plain text password, encode it, and set it on a new password property.

That means we need a place on a user where the plain text password is stored, so create another property called "plan password". Do not persist this in the database, so do not put at or am slash column, this is just meant for storage inside of our application. Then at the bottom, I'll go down and generate the getters and setters plain password as well. Then let's set plain password. I want you to say this arrow password equals null. This is very important. Because we're going to have a doctrine listener automatically change the plain password into a password, that only happens when doctrine detects that the user object has changed in some way. This is a necessary step to avoid a bug where if you allow the user to only change their plain password, doctrine won't run the [MLSname 00:03:41]. That's what happening in there.

It's a necessary evil. In erase credentials, I want you to say that this arrow, plain password equals null. That's a really subtle thing, but that's something the security system calls before it puts your user in a session. It just makes sure that we don't accidentally end up with a plain password stored in the session somewhere, which we do not want.

The user object is looking perfect, so let's add our doctrine listener. In AppBundle, create a new directory called "Doctrine." Then a new class inside of that called HashPasswordListener. If you've never done a doctrine listener before, they're pretty simple. You're going to implement an event subscriber interface, and then I'll go to Cogenerate, go to Implement Methods, and I'll select the one here, which is GetSubscribedEvents. It's very similar to Symfony event subscribers. There's just some subtle differences. In here, we're going to return an array with prePersist and preUpdate, because of course we're going to want to encode the password on prePersist. That means one user is originally saved, and also on preUpdate, because we might change the plain password later.

Next we'll make public function prePersist, and whenever you have a listener doctrine, you're past a LifeCycleEventArg, so this is just from the doctrine ORM namespace. This is important, because it gives us access to the entity that's being saved, because this function will be called whenever any entity is being saved. We'll say entity equals args arrow getEntity, then if not entity instance of user, then just return. We don't actually want to do anything on that.

In order to encode the password, Symfony gives us a built in service to take care of that called User Password Encoder. We're eventually going to register this to the service. I'll make the constructor, use this user password encoder, and then I'll hit option enter to initialize that field and save me a little bit of time. Here, we'll say encoded equals this arrow password encoder arrow encode password. Then we're going to pass it to the user, which is actually entity, and then we're going to pass it the plain password, which we know is going to be entity arrow get plain password. Perfect. Then, ultimately, we'll say entity arrow set password. That's it. Encoded.

The document will continue saving, and it will save that password. Cool. This is all we need for this, but let's also take care of free update. Free update's going to be almost the same thing, so I'm going to select these two lines here, go to command T for the Refactor This menu, put on the menu, and we're going to have this create a new, private function called Encode Password. I get one argument that is [type printed 00:07:58] the user class. You can see it's just going to take those two lines and create a nice private function down here on the bottom. You could have also done that by hand if you're not using [Page restore 00:08:07].

Now we have that, copy prePersist, preUpdate, and they're almost the same. The beginning's the same, the encode password's the same. There's just one little hack you need in preUpdate which is due to a little doctrine port, what you actually need to tell doctrine "I have updated this entity, so I need you to actually look and notice the changes." This is only needed in preUpdate, and it looks like this craziness. EM equals args get entity manager. Meta equals EM arrow get class metadata, get class entity. Then, EM arrow get unit of work arrow recompute single entity change set. Pass that to meta, and you pass that the entity.

What that does is it tells doctrine to notice that you just changed the password field so it'll save it. You leave out those last 3 lines, then your new, encoded password won't actually get saved.

Event subscriber's perfect. The last step is to register this as a service and hook it up so doctrine notices it. In AppConfig services.yml, we'll create a new service called app.doctrine.hashpasswordlistener. Set up the class, of course you guys know at this point I like to autowire things. Doesn't always work, but it works in most cases, as you can see. Then, to tell doctrine about our event subscriber, we're going to add a tag. I'm just using a little shortcut I have set up for that. You'll say doctrine.event_subscriber, not listener. We've set ours up as a subscriber, and that's it. The system's complete.

Now that we have this happening automatically, we can go to our fixtures.yml file and down where I have our users, we can just add one for plain password. We'll keep using Iliketurtles. If all goes well, when we load our fixtures, it should automatically create an encoded password and save it on the password field for that password.

Let's flip over. We're on bin console doctrine fixtures load. This will not work yet. Check this out. No encoder has been configured for AppBundle Entity User. What this is basically saying is, "Ryan, you didn't tell me how you want to encode the passwords." I keep saying we're going to use bcrypt, but there are many ways to encode passwords, and something that you need to configure with Symfony so it knows how to do it. This is really simple. It's in security.yml. You just need to add an encoder scheme, followed by full name space to your user class, and then which algorithm you want to use. If you want, there are a couple other options you can pass to that, but that's good enough.

I'll switch back, load the fixtures again. This time, no errors. That probably worked, so let's try it. In login form authenticator, we can finally get rid of our simple password checking. In order to ... What we want to do is compare these submitted plain text password, encode it in the same way, and see if it matches the value we have in the database. Fortunately, that same user password encoder can check the password too.

Go back up to construct, type in user password encoder, I'll hold option enter to initialize that field. Down here, we can replace this if statement with if this arrow password encoder arrow is password valid, we'll pass it the user object that will use it to fetch off the encoded password. Then, we pass it the plain text password, and that will take care of everything else for us.

Let's try it out. Go to login. Get our user. Iliketurtles. There is is. Password system check.
