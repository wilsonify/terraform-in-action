# What is Terraform

Terraform is an infrastructure management tool made by HashiCorp that lets you provision, manage and maintain Cloud
resources, like servers, networking, storage, et cetera, all in one centralized set of code.

So Terraform is a tool, command line program you run to define and make changes to your infrastructure.

But Terraform is also a language that defines those changes.

Terraform is for managing the base infrastructure itself.

For example, creating a server instance and placing it behind a load balancer.

Terraform can't really change what's running on that server once it's deployed.

For that you'd need a config management tool or some other method.

So Terraform will let you create the server and then you might use something like Puppet to turn that server into a web
server with your specific application code running on that.

You might think of Terraform as setting up a blank canvas.

But you'll still need other tools to paint the picture.

It is possible to run a tool like Puppet on a system using a Terraform feature called Provisioners.

But we're intentionally not covering them in this course.

The official Terraform docs describe Provisioners as a last resort. And I agree.

Although I can see how they could be amazingly powerful if you're already using config management.

We don't want to set them up right from the beginning.

So if we won't use config management, what's our other option?

Terraform is really well suited to deploy pre made server images.

Where all the configuration is either already set or is looked up at run time.

You'll still need to create a base image and there's another great Hashicorp tool for that called Packer.

If you set things up correctly with a load balancer and an application that can tolerate cycling server instances, you
can use Terraform to maintain and update systems by destroying and replacing them with new instances.

The purest form of this approach is called immutable infrastructure.

Where even the system disks are read only and config and software can only be updated by replacing the instance.

Even if you don't go to that extreme, being able to freely throw away and replace any instance is a total game changer.

For me that leap has drastically reduced my background anxiety around site stability and on-call shifts.

Finally, we get to containers. Terraform is able to manage these.

Terraform can work directly with Docker and Kubernetes offering a couple of approaches for container-based workload.

Kubernetes could also be paired with say, Amazon's Kubernetes support or through another cloud provider.

You could even use Terraform to provision a Kubernetes cluster on a Cloud provider.

And then use the Terraform Kubernetes provider to provision containers on that cluster.

# Initial deployment

Now that we've got that initial code written, we're ready to actually try out Terraform.

So first we'll need to run the Terraform init command. And that's literally just:

```
terraform init
```

And that needs to be run in the same directory, as that dot tf file we just created.

This will initialize Terraform, and it will look for any .tf files, it can find in the local directory.

So, it should see ours, pick up the fact, that we're using A-W-S, and then automatically download, and install the
additional Terraform plugin it needs, for working with A-W-S.

If we had other providers to find, it would find them, and download plugins for them at this point.

So, once that's done, you're really ready, to apply this code, and the command, in this case is ```terraform apply```.

And I know we're going kind of quickly through this, I just want to point that out.

We're going to circle back to all these commands, and to the code, and syntax, and go through it in detail later on.

The point of this is just to get a taste, of what Terraform is like, so that you can see how powerful it is.

So, I'm just going to run ```terraform apply```.

And you may or may not see a message, about setting your A-W-S region.

If you do, just follow those instructions.

I didn't, because I've already run through it.

And if you've already used A-W-S for something else, you may not see it then, because it will already be configured.

So, now we've run our apply, and Terraform has given us this output, which is actually a detailed description of, what
it's going to do.

It's saying an execution plan has been generated, and is shown below.

It's going to create this resource.

This A-W-S S-3 bucket that we've defined.

And here's the various details, and a lot of them are not known yet, they'll be known after it's actually been created.

So, it's asking us do we want to perform these actions.

I'm going to say yes, and hit enter.

Now Terraform is actually going out, make those changes and you can see it's done.

It happens pretty quickly.

If you run into a credentials error here, go ahead and double check that A-W-S credentials file.

A simple typo in that file, will cause Terraform to throw a no valid credentials error, and it can be a little bit
cryptic.

Also make sure that the file is just called credentials, with no file extension.

Notepad will save it by default as a .txt file, so you might need to rename it.

If your still struggling with that point, I'd recommend, pausing this course, and going and finding a course on A-W-S,
where you can just kind of learn the details, about how to do that basic C-L-I setup.

This is kind of standard A-W-S command line config, and so anything you learn in another course, should cross-apply in
this case.

And that's really it for our initial deployment.

As I said, we'll dig in deeper to what's happening later on.

But I hope this shows you, just how powerful Terraform can be.

With that tiny bit of code, and just a couple of simple commands, we're already up and running, and managing a resource
on A-W-S.

# How Terraform works

At first glance, what Terraform does might not seem that complex.

You're able to write some code to define some cloud resources, run a program, and it goes out and makes those things
happen.

So, what is it that makes Terraform different from just writing a simple script that calls out to the AWS API?

Terraform lets you define you're infrastructure as code, and gives a lot of flexibility in how you do that because you
can freely use data from one resource to define another.

In a non-Terraform script that just uses the API, you might deploy a couple of web servers and a load balancer.

If you also want to add a new security group to function as a firewall, your script will need to call out to AWS,
retrieve and process information describing those resources, and then take action to define your security group.

With Terraform, that kind of sharing of data is trivial.

Your code can define resources based on the definitions of other resources, even if they don't exist yet.

That last point is critical.

Terraform figures out the hard part of resource ordering and lets you just treat the infrastructure as static code.

One thing that I found a bit intimidating at first about Terraform, was the idea that it can just go and make huge
changes in my production infrastructure.

Potentially deleting critical components and causing a serious outage.

When I got more comfortable with the tool, I realized just how powerful the execution plan is.

There is no need to worry about Terraform making a change you don't expect.

When it compiles the plan, you'll see exactly what Terraform will attempt and have a chance to decide if it's what you
actually want to happen.

So, Terraform is taking the infrastructure described in your code and comparing it to the state of what actually exists,
and then essentially writing a step by step script that will make the changes.

The plan step is critical because it's the bit that figures out what needs to be done and in what order.

Then, Terraform uses the provider to actually apply the plan and make whatever changes are needed.

Terraform is able to figure out how to do these things by using a data structure that's known as a graph.

More specifically, it uses a type of graph called a Directed Acyclic Graph.

That sounds really technical and complicated, but it's actually a simple concept if you unpack the name a bit.

First of all, a graph is made of a series of connected nodes, like these three circles connected by lines.

In the case of Terraform, each node is a resource like an S3 bucket, a domain name, or an EC2 instance,  
plus some others that Terraform uses internally to keep track of things.

The graph is directed, that means there's an order.

So, in our image, we'd replace the lines with arrows to show direction.

If we add in an arrow back from node three to node two, we could say that we've created a cycle.

You can loop back and forth between two and three by following the arrows.

For a graph to be acyclic, it means that you can't have any cycles like that.

If you think about it, it makes sense in terms of how Terraform uses the graph.

You don't want to create a resource, move on to another one, and then come right back and recreate the first one.

It means that you'll sometimes need to change the logic and ordering of your code to make sure that you don't have two
independent resources that form a cycle.

This connection, on the other hand, is totally okay because it gives Terraform more information about the order things
can happen.

A real world Terraform run will generate a much more complex graph with lots of interconnections.

That data structure is how it sorts it all out under the hood.

# Terraform plan

So we've taken a high-level look at how Terraform works, but let's get our hands dirty again and try some of these
things out.

Creating the execution plan is a critical step in Terraform's workflow, and one way to trigger that is with the $
```terraform plan``` command.

It will also happen automatically with the $ ```terraform apply``` command, as it did in our first exercise, where we just
tried deploying a resource.

Let's take a look at the built-in help so we can see what options are available with $ ```terraform plan```.

So the command is just $ ```terraform plan``` --help.

That help flag will work with any of the Terraform commands.

It's a very handy way to see the syntax.

So just take a minute now, maybe pause the video, and look through some of these options, and some of this documentation
here.

We're not going to go through all of these, we're just going to look at some of the most important ones, but I just want
to start with the command itself, with no other options defined.

So I just say $ ```terraform plan```, and again, we're in our same code directory where we defined that s3 bucket earlier.

So the command is just $ ```terraform plan```.

Terraform is good about telling you what is going on as it happens.

So first you can see it refreshes the state, and then it's explaining that it's just in memory and not persistent.

That is, it's not saved to a local file or something like that.

Then you can see it actually calls out to AWS to retrieve the state of the bucket, and finally, we have this section at
the bottom where it says it doesn't need to do anything.

That's 'cause we already applied our changes.

If you didn't happen to actually run $ ```terraform apply``` in the earlier lesson, you should see a plan described here to
create some resources.

So let's look at some of the command line options.

First, we can see the inverse of our plan by using the destroy flag.

So I'll say $ ```terraform plan -destroy```.

Now remember, this is just a plan, it isn't actually going to take any action here.

So let's hit enter. So there's the result of that, and let's scroll up and take a look.

You can see it's planning to destroy my bucket.

So you can see it's got all these details here about what the bucket is, and some of its ID numbers and things like
that, as they exist currently on AWS.

So let's go back to our prompt.

There's a note here about specifying a -out parameter.

You remember how when we first deployed our code, how we just ran $ ```terraform apply``` without running plan first? In that
case, Terraform still generates the plan for you, and then it just uses it immediately.

If we ran apply now, we would actually need to re-generate this plan, which means it might not be exactly the same as
what we just generated.

So setting the -out parameter lets you separate plan and apply, and it means you can trust that exactly what it says
will happen is what it will try to do.

So let's go ahead and try that out right now.

$ ```terraform plan```, and I want to say -destroy here, because I already have the resource created, and I'll just make an
out file called example.

plan, and so it's -out= and then the name of the plan.

I don't know if there is a proper file name here, but I use .

P-L-A-N as my format.

So I'll just hit enter there, and it's generating that plan.

It displays it for us, but you can see at the bottom, it's also saved it to that file.

And now we can inspect the result of that by saying $ terraform show, and then the name of our plan, example.

plan.

And there you go, you can see it's the same thing that's displayed up above, as it's generated.

By the way, in case you're wondering, this plan file is a binary file, so you'll need to use Terraform to inspect it.

If you try to just cat it out, you'll get this.

(computer error chime) So that's the basics of $ ```terraform plan```.

We'll see how this fits in a little later on as we learn $ ```terraform apply```.

# Terraform state

You'll often run into the term state when working with Terraform. It shows up even in the output of the command line
application. But what exactly does state mean for Terraform?

By and large, Terraform State means what it sounds like, the state of your resources in Terraform.

But it's important to realize that it's used to refer to two different things.

One is the reality of your infrastructure.

What is the state of the resources on AWS? Their IP addresses, their instance types, bucket names, et cetera.

It also refers to the local representation that Terraform keeps, called the state file.

It's entirely possible for those two to be out of sync, and Terraform doesn't know until it refreshes its local state.

Let's hop in to command line terminal and take a look at the state file.

You can inspect the state file from the most recent Terraform run directly.

It's a JSON format file called terraform.tfstate.

Let's take a look at it.

I recently ran a ```terraform apply delete``` and cleared out everything in my infrastructure.

Let me go ahead and run apply and I'll reprovision everything.

Now, the code I'm using right now is actually from a later point in the course, just 'cause I wanted a little more to
show you.

So, ```terraform apply```.

And I'm just going to say yes.

You can see it's provisioning quite a bit of stuff, a couple of instances, a load balancer, and then, these things we've
already provisioned.

Let me just run ```terraform plan```.

And let's keep an eye on what the messages are.

You can see it says at the top,

    "Refreshing Terraform state in-memory prior to plan.
    The refreshed state will be used to calculate this plan,
    but will not be persisted to local or remote state storage."

Now, Terraform state is stored locally on our system in the tfstate file.

So, let's take a look at that before we talk anymore.

I'm just going to open it up in vim.

So, terraform.tfstate, but it's actually a JSON file.

This is an internal file used by Terraform.

It is possible to get in here and mess with this, but I'd actually recommend you do whatever you can to avoid that
because it just is very hard to get it right.

But this is essentially all the details about our infrastructure.

So, you can see each of the resources that are mentioned, there's details about providers, essentially, anything that
Terraform knows is right here.

And this is what it thinks the current state up on AWS is.

Let me quit this file.

So, that's the local storage.

It also mentions remote storage.

Remote storage is something you'd have to set up if you were working on a team using a feature Terraform call backends.

Remote storage of state prevents a team from stomping on each other's changes and allows teams to do things like
delegate and share resources.

For example, you could allow some users read-only access to certain details of the infrastructure, but still let
Terraform run as though it had full access, as long as it didn't change this.

So, let's look at actually the preferred way of investigating Terraform state, and that is the terraform state command.
So, this command has a few options that you can see here in the description. And some of 'em are intended only for
advanced cases.

It actually says that.

    "This is sometimes necessary in advanced cases."

As I mentioned, I'll attest that messing with the state and the state file is something I try to avoid at all cost. It
is needed sometimes.

Sometimes things can get just a little bit out of sync, and Terraform can't recover itself.

But that is an advanced thing on a large infrastructure and messing with state shouldn't be done lightly.

The read-only sub commands are totally find, though.

So, the first one is terraform state list.

And you can see here it's listed out just the types and names of all the resources that we have provisioned right now.

And as I said, I'm later in the course, so, there's a few that you haven't set up yet.

But if you're following along, you should see a couple of these.

And we can dig in a little deeper into one of these resources with the command terraform state show and then, the name
of the resource.

So, let me look at, say, my security group, aws_security_group. prod_web.

And here's all the details of the security group.

Now, this includes things that are settings I didn't set.

They're settings that were provided by Amazon.

And although this response is essentially Terraform code, those settings that were provided by Amazon are included in
here too.

So, if you were going to use this and try to use it to create another instance of one of these resources, you'd want to
clear out that stuff.

So, that's like the ARN and the ID number, and a few other things like that, the owner ID.

Some things you can provide and it'll just use the same setting that's already there but actually, I think it's better
to take those things out of your code and make it as generic as possible.

So, this command is a pretty handy way to get information about the state without having to wade through that state file
or just log in to the AWS DOI and try to find things in that way.

If you do want to see the entire state file, you can actually run terraform show, which uses that same format but it
just dumps out every single one of those resources.

And if you want to use this in a programmatic way, like in a wrapper script for your Terramform binary, you can use
-jason.

And that'll give you the whole thing in a big JSON blob here.

In real world usage, it's pretty common to use a wrapper script around Terraform.

So, being able to get machine-readable output of the state is a really nice feature.

Also, if you're savvy with JQ or a similar tool, it can make a really handy way to quickly query your state.

And if you haven't heard of JQ, I just want to put in a little plug, it's one of my favorite command line tools, it lets
you essentially write queries against JSON objects.

It's super handy.

And that's Terraform State.

# Terraform graph

If you're more of a visual interactive learner like me, I think you'll appreciate this lesson.

Because Terraform builds a graph as part of its plan, we can actually get some insight into what's going to happen by
exporting that graph and rendering it visually.

The command to do this is just Terraform Graph.

Now, take a look at this output.

This syntax is known as DOT for DOT file.

It's a pretty common way of defining graphs like this and looking at it even if you haven't really encountered the
syntax before, it's possible you could figure out what's happening but there's a better option than trying to understand
this text format.

So, go ahead and open a web browser and the site is webgraphviz.com.

This is a web-based tool for generating a visual graph from the DOT format file.

By the way, the underlying tool, Graphviz, is free on Open-source and it's really pretty easy to set up.

So, you could just use that directly if you prefer.

It lets you take this text format and output an image file.

Before we generate the graph from our Terraform code, let's just take a look at one of these examples.

Now, this one that came up when the page loaded is pretty simple and I think if you look at it, you might be able to
figure out what it's going to look like.

Just take a guess if you think you know and then click Generate Graph.

So, you can see it's got four nodes and they're connected to each other in this particular order and maybe if you look
at that, you can understand a little bit of this format.

It's not that important to actually understand it. Now, let's head back to our terminal and I'm just going to copy this
and I'm going to paste that code back into the web browser and click Generate Graph again.

So, let's scroll down and take a look at our graph.

As you can see, even though there's just a single S3 bucket in our code, the graph is not trivial.

It begins with a route node.

That's just so Terraform has a place to start basically and then you can see there are two resources defined below that.

Don't worry too much about the details.

This is mostly just internal stuff used by Terraform but the important thing to notice here is that these are parallel.

So, our graph is still a cyclic since it can't loop back but both of those nodes happen essentially at the same time.
When we get to more complex code, you'll see that this can happen with resources and have a better sense of
understanding how different resources can be balanced and ordered.

So, after those two parallel nodes, there's our bucket and you can see here the way Terraform thinks of it internally,
which is the resource type and then a DOT and then an actual name that we assigned, which in this case was TFCourse.

The last element in our graph here is this diamond shape, which is just provider. aws and, again, that connects with
this and what it's saying here is that all of that information is getting handed off to that provider and then the
provider is going out and actually making it happen on AWS.

If we had multiple providers, say we also had Azure Resources or Google Cloud or something like that, they would also be
represented in this graph as diamond shapes in the same way.

# Terraform apply

We ran ```terraform apply``` early on just to kind of get things going, but in this lesson, we're going to dig a little bit
deeper.

To begin, let's go ahead and regenerate that plan and then apply it using an output file.

So first you'll need to do a destroy plan since the resources have already been created.

So ```terraform plan``` dash destroy dash out equals example. plan.

So now that we've generated that, we can run ```terraform apply``` and then just example. plan.

You'll notice that this time, unlike when we first ran it, we weren't prompted for a yes/no response.

When you split them out like this, Terraform assumes that you want to go ahead and apply the changes that you just told
it to apply.

Now, you might be wondering what happens if you run the same apply again, so let's try it.

We're just going to apply the exact same thing.

As you can see, Terraform knows that this plan is stale so it won't let you reapply.

The resources have already been deleted, so running this wouldn't do anything anyway.

Now let's re-create our bucket with just ```terraform apply``` this time.

Now here it's prompting me for a yes or no, and let's just take a look quickly at the plan it's generated.

And I'll go ahead and say yes.

Now, you might imagine at this point you could apply that delete plan again, so let's see what happens when we do that.

You can see it's still marked as stale.

If we want to delete these, we need to generate a totally new plan.

That plan to delete that previous resource is no longer relevant to what we have right now.

Let me clear my screen so we can just see a bit more.

And now let's regenerate that plan again.

So ```terraform plan``` and then destroy, and we use that same output file.

So here it saved it and we can apply it again.

So now that bucket's been deleted, so we could apply this plan again.

This time I want to just show one other flag which is the auto-approve flag.

So we'll say ```terraform apply``` dash auto dash approve.

And that's just going to do back to back our plan and apply without prompting us.

You can see it doesn't even actually output the plan result.

Now, obviously you should be very careful about auto approve, but it's nice in times like this when we're just trying
things out or we just want to quickly undo something we just did.

 