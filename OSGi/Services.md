# Services in OSGi

OSGi offers a Services registry where services can be found, registered and unregistered. This registry functionality is not present in the bare Java platform. 

We'll explore the advantages of Services by making a simple anagram game where you have to guess the scrambled word.

- Challenge: `rrokoigs-femaw` 
- Answer: `osgi-framework`


We'll describe how you can make this project yourself. For your reference, you may find the complete workspace that contains this example in the following [Github repository](https://github.com/jpwijbenga/osgi-tutorial).

## Creating the garbler  API
Creating an interface (or API) is a good software engineering practice to achieve decoupling the specific implementation from the generic functionality. The interface serves as the agreement between the calling code and the implementing code. The caller sees only the interface, and in this way the implementing code may be optimized, improved or replaced as long as the interface is adhered to.

In this project we will make a service that mixes the letters of words: the garbler.
We start off by making an interface. It merely states that any implementer should have the method `garbleWord` accepting a String object and returning a String object.

Start off by starting a new workspace and adding the special `cnf` folder. This was described in the previous chapter.

The next step is to create a new bundle for our API called `org.flexiblepower.tutorial.api` (`File` -> `New Bndtools OSGi project`).
Eclipse may give an error regarding the `res` folder. This is solved by right-clicking the `org.flexiblepower.tutorial.api` project and adding a new folder called `res` to the root of the project. Also the template may have added `osgi.annotations` to the build path (look for it in the `build` tab of the `bnd.bnd` file), so let's remove this unused bundle. Let's add the `osgi.core` bundle as well for our service to be ready to use the OSGi functionality we'll be needing.

Now we'll add the interface definition to this project. Add a new interface to the `src` folder called `GarbleService.java` and define the interface using one public method that takes a String returns a String and is called `garbleWord`. The package name may be the same as the bundle name: `org.flexiblepower.tutorial.api`. Since we don't use many classes there is no need to make different packages in here.   

## Creating a garbler service

Now that we've put in place the API, let's make an actual implementation. We'll have to add the implementation to a separate bundle to nicely separate the interface from the implementation. Let's call the new bundle `org.flexiblepower.tutorial.services` and set it up in the same way as the API project (as described in the previous paragraph).

Now we'll add the API bundle to the build path (using the `build` tab of the `bnd.bnd` file of the `services` bundle). Only after we've done this, we can reference the API from code.

Now let's construct an implementation of the interface. Right-click the `services` project and add a new class `ShuffleGarbler` (sometimes also the `-Impl` suffix is used to distinguish interfaces from implementing classes). When constructing the new class through the dialog in Eclipse make sure you name the package `org.flexiblepower.tutorial.services.garbler` and don't forget to define the interface `GarbleService` (the interface name should be found and auto-completed because we've added the reference to the API bundle).

Eclipse will say that you've yet to define the method that implements the interface functionality. Create the overridden method and add the following implementation:
```
		List<String> solution = new ArrayList<String>();
		for (String s : word.split(""))
		{
		    solution.add(s);
		}
		Collections.shuffle(solution);
		StringBuilder builder = new StringBuilder();
		for(String s : solution) {
			builder.append(s);
		}
		return builder.toString();
```

Even though this is an implementation of the interface, this is not a functional bundle yet. We've yet to define a `BundleActivator` class that tells OSGi which methods to call when starting and stopping this bundle. So let's add a separate class called `GarbleActivator` in the same package as the `ShuffleGarbler` that implements only the `BundleActivator`. The `start` method should make a new `ShuffleGarbler` object. OSGi offers the ability to add a property as extra information about the specifics of the service. We'll use the property to say that our garbler handles all sorts of strings (some other garblers may only accept numeric strings for example). The properties are stored in a `Hashtable`. The next piece of code shows the body of the `start` method of this bundle activator.
```
		ShuffleGarbler garbler = new ShuffleGarbler(); 
		Hashtable<String, String> props = new Hashtable<String, String>();
        props.put("CompatibleStrings", "All");
        props.put("GarblingType", "Shuffle");
        context.registerService(
            GarbleService.class.getName(), garbler, props);
```
## About Service properties

The properties allow you to differentiate between different services of the same service type. In this case, we choose to add two properties:

- Which strings is this garbler able to handle (could also be for example numeric or only fixed-length).
- How this service garbles words (could also be for example reverse or insert garbage characters).

Any runtime service may filter on these properties to find the right variety of the service in this way.

## Print service events with the  `ServicePrinter`

As a side step, we'll create a `ServicePrinter` bundle that prints service registration and unregistration events. Create a new bundle as described in the `A new Bundle` section of the `Bundle` chapter of this tutorial. Again, we need to make sure we reference only the `osgi.core` import and we need to create the empty `res` folder in the project root folder. These steps should get rid of any error messages Eclipse might give.

The bundle and package may be called `org.flexiblepower.tutorial.serviceprinter` and the class `ServicePrinter.java`. We need this class to implement the `BundleActivator` interface, so that the OSGi framework can start and stop it. Also, we need to implement an interface called `ServiceListener` to allow us to listen for services. The start and stop method adds and removes the service listener to the `BundleContext` object. The `BundleContext` and underlying classes provides the rest of the functionality. The `ServiceChanged` event looks for registered, unregistered and modified services and prints it to the standard `out` stream. This is an example of how the total class may look after these steps:

```
package org.flexiblepower.tutorial.serviceprinter;

import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.osgi.framework.ServiceEvent;
import org.osgi.framework.ServiceListener;

public class ServicePrinter implements BundleActivator, ServiceListener {
	
	@Override
	public void start(BundleContext context) throws Exception {
        System.out.println("Starting to listen for service events.");
        context.addServiceListener(this);
	}

	@Override
	public void stop(BundleContext context) throws Exception {
        context.removeServiceListener(this);
        System.out.println("Stopped listening for service events.");
	}

    /**
     * Implements ServiceListener.serviceChanged().
     * Prints the details of any service event from the framework.
     * @param event the fired service event.
    **/
	@Override
    public void serviceChanged(ServiceEvent event)
    {
        String[] objectClass = (String[])
            event.getServiceReference().getProperty("objectClass");

        if (event.getType() == ServiceEvent.REGISTERED)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " registered.");
        }
        else if (event.getType() == ServiceEvent.UNREGISTERING)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " unregistered.");
        }
        else if (event.getType() == ServiceEvent.MODIFIED)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " modified.");
        }
    }
}
```
Finally, in this project's `bnd.bnd` file we need to indicate that this class is the `BundleActivator` for this bundle. This can be done in the `Contents` tab. After this, OSGi knows how to activate this bundle.

## Creating a run descriptor.

We should now have a workspace with four projects: a `cnf` project, an `api`, a `services` and a `serviceprinter` project. Note that the last three projects are OSGi bundles and the first one is not a bundle, but contains the shared information about the build process and repositories.
Add a `New` -> `Run Descriptor` to the cnf folder. You may use the empty Apache Felix 4 template as a starting point. Create the run descriptor, open it and switch to the `Source` tab to see what's in there. Check if the versions of your Java and Felix are in order. 

There are `runbundles` and `runrequires`. The difference is that the `runrequires` bundles are selected by you to indicate which bundles you want to run. OSGi can then resolve and add the set of additional bundles that are necessary to run our selected bundles. The total list is the `runbundles` list. Switch back to the `Run` tab to see the difference between these two lists.

You'll see that the `runrequires` list does not yet reference our newly created projects. Nor does it contain the `webconsole` and `Jetty server` bundles that we'd like to have for our demo. The web console and the Jetty server will provide an out of the box osgi dashboard, so that's a nice flying start for our demo. The downside is that they require a lot of bundles to run, so don't be scared when you'll see a large list of bundles later on in the demo.

From the `Available Bundles` sub window we'll add:

- `org.apache.felix.webconsole`
- `org.apache.felix.http.jetty`
- `org.flexiblepower.tutorial.services`
- `org.flexiblepower.tutorial.serviceprinter`.

Also we'll check the `Auto-resolve on save` box. Now we'll save the changes to the run file. Next, it should indicate correct resolution of the bundles. Note that we did not have to reference the API bundle as a run requirement; the resolution process determines this bundle is necessary and adds it to the `runbundles` list.

## Running the project

At this point we should have a workspace with four projects and a run descriptor to start up everything. We don't have the implementation of the anagram game but we have an api and a service, plus a service listener. Let's try to see if we can do a run.

With the run descriptor open on the `Run` tab we can click `Debug OSGi` and our bundles should start. The Console window in Eclipse shows the output of the HTTP server and also should already show the service registration events. Our `ServicePrinter` prints all service registration events that were captured since the `ServicePrinter` process started.

Next we open the web console (`http://localhost:8080/system/console/bundles`). The default user name and password are `admin` and `admin`. We'll see that our bundles are `Active`, if all went well. Our `ServicesPrinter` program already printed some service activity, but it doesn't display all services. Luckily, the web console offers the `Services` page (found in the menu under `OSGi` -> `Services`). You'll see there are a lot of services running; most of them belong to the Felix OSGi framework and the web console itself. If you look closely, you should see our custom `GarbleService` under the specified name. Clicking the arrow in front of the `ServiceId` shows the service properties. Click the arrow next to `GarbleService` to check the service properties we've added to the service description. Now you may terminate the debugging session.

The fact that our service was showing in the web console, means that any bundle can now reference this. Let's create the anagram game that uses this service.

## Finding and using a service

Let's start by making a new project or, more precisely, a new `Bndtools`  OSGi project that results in a bundle. Preferably specify a Java version of `SE7` or later because we'll be using some functionality of this edition. Go to the `bnd.bnd` file and to the build path bundle list. Make sure to add the `osgi.core` version 5 or higher, because we'll be using the generic `ServiceTracker` class included in these versions. Next, we add the API bundle to the build path. These are the only two bundles you should reference, so remove any other reference that the template may have added.
Call the bundle `org.flexiblepower.tutorial.games` and add an activator `AnagramGameActivator`. You'll see that we run the game in a new thread. (Later on we'll learn about Components as a nicer way to do this compared to using a thread.) If we don't start the game in a new thread OSGi will get stuck in the start method of this bundle, and stop loading additional bundles. This would clearly not be a desirable situation so we start a new thread that allows for continuing to start the other bundles.

Note that this example does not provide an elegant stop method yet (this is left as an exercise for the reader ;-)).

```
package org.flexiblepower.tutorial.games;

import org.flexiblepower.tutorial.api.GarbleService;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.osgi.util.tracker.ServiceTracker;

public class AnagramGameActivator implements BundleActivator {

	private AnagramGame game;
	private Thread thread;
	private ServiceTracker<GarbleService, GarbleService> tracker;
	
	@Override
	public void start(BundleContext context) throws Exception {
		System.out.println("The Anagram Game bundle was started.");
		tracker = new ServiceTracker<GarbleService, GarbleService>(context, GarbleService.class, null);
		tracker.open();
		
		game = new AnagramGame(context, tracker);
		thread = new Thread(game);
		thread.start();
	}

	@Override
	public void stop(BundleContext context) throws Exception {
		// This sets the flag of the game to terminate.
		game.terminate();
		// Close the tracker.
		tracker.close();
		System.out.println("The Anagram Game bundle was stopped.");
	}
}
```
You'll see an error message about the unresolved type `AnagramGame`. Using the suggestions from Eclipse's error message you can create the `AnagramGame` class. It should implement the `Runnable` interface. Place the `AnagramGame.java` in the same package as the activator.

We'll add the implementation of a game that finds services using the `ServiceTracker<S,T>` object. Make sure your `osgi.core` reference in the `bnd.bnd` file has version 5.0 or higher, because this class was moved to the core at a certain point in time. The `ServiceTracker` needs two type parameters to specify the tracked object's original type and the type you wish to receive from the `ServiceTracker`. This tracker is created and should be opened in the constructor. If it's not opened, it's not tracking anything. The run method first searches for the `GarbleService` using the `ServiceTracker` method: `GarbleService garbler = tracker.getService();`. If none is found, null is returned, so we need to check for this. Because we don't know which bundle will start first, we'll add a waiting loop with a delayed retry.

Note that we start and stop the tracker in the activator. 

The game itself garbles a randomly chosen word and asks the user to input the answer. After one try the game is over and you'll have to start it again to retry. The word challenges are contained in a string array called vocabulary. The constructor starts the buffered reader for reading console input and sets the service tracker reference for finding a garble service. The `run` method first waits for a garble service to become available. After that a loop is started that starts a new game with a new challenge. The flag `running` keeps track of the stopping signal.

```
package org.flexiblepower.tutorial.games;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Random;

import org.flexiblepower.tutorial.api.GarbleService;
import org.osgi.util.tracker.ServiceTracker;

public class AnagramGame implements Runnable {
	// The anagram challenges.
	private String[] vocabulary = { "crossword", "osgi-service", "flexiblepower", "powermatcher", "alliance",
			"anagram" };
	// This object can look up the right service to garble our words.
	private ServiceTracker<GarbleService, GarbleService> tracker;
	// The reader object.
	private BufferedReader reader;
	// The random number generator to pick a random word.
	private Random rnd = new Random();
	// Flag to indicate if we're still running. It is marked volatile for thread
	// safeness.
	private volatile boolean running = true;

	public AnagramGame(final ServiceTracker<GarbleService, GarbleService> theTracker) throws InterruptedException {
		tracker = theTracker;
		reader = new BufferedReader(new InputStreamReader(System.in));
	}

	@Override
	public void run() {
		// First we wait for a garble service to become available.
		System.out.println("Searching for the garble service...");
		GarbleService garbler = tracker.getService();
		while (garbler == null) {
			System.out.println("Garble service is not online yet. Retrying in 3 seconds...");
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
				running = false;
			}
			if (!running) {
				// If the stop method was called by OSGi we should stop running.
				return;
			}
			garbler = tracker.getService();
		}
		
		// The garbler service is online. Now we can start the game.
		try {
			while (running) {
				System.out.println("Picking a new challenge.");
				String challenge = vocabulary[rnd.nextInt(vocabulary.length)];
				startNewGame(reader, garbler, challenge);
			}
		} catch (IOException e) {
			e.printStackTrace();
			System.out.println("IOException reading the input.");
		}
		// Program runs out of scope and thus exits here.
		System.out.println("Quitting now.");
	}

	private void startNewGame(BufferedReader reader, GarbleService garbler, String challenge) throws IOException {
		System.out.println("Garbling a word now...");
		String garbled = "";
		if (garbler != null) {
			garbled = garbler.garbleWord(challenge);
		} else {
			System.out.println("Can not find garbling service. Closing...");
			return;
		}
		System.out.println("What's the hidden word in this anagram? (type and press Enter)\n" + garbled);

		String input = reader.readLine();
		if (input.equals(challenge)) {
			System.out.println("Correct! You've found the answer.");
		} else {
			System.out.println("Incorrect answer. The correct answer was " + challenge + ".");
		}
		if (!running) {
			// The OSGi framework called the stop method
			return;
		}
		System.out.println("Type 'quit' <Enter> to quit or just hit <Enter> to start again. ");
		if (reader.readLine().equals("quit")) {
			running = false;
		}
	}

	public void terminate() {
		if (running) {
			System.out.println("  <<OSGi called stop method so gracefully shutting down soon>>");
		}
		running = false;
	}
}
```

## Running the game

The game bundle `org.flexiblepower.tutorial.games` is not yet listed in the run requirements in the run descriptor. Therefore our game will not start when the run descriptor is being executed. Let's go to the bndrun file that we created in a previous section. It should be in the cnf project. The `Run` tab shows a list of available bundles, including our game bundle. Let's drag `org.flexiblepower.tutorial.games` to the Run Requirements window. Next a save action will trigger the reolution if you've checked the `Auto-resolve on save` option. Finally, click Debug OSGi to execute the run file.

You can check out the web console at the [default address](http://localhost:8080/system/console) and browse through the tabs to see if you understand what's going on.

The console of Eclipse will show the combined output of the bundles. There is some server logging originating from the web console, some logging of our service listener bundle and there should be the output of the anagram game. If all went well, you should be able to play the game.
The game allows you to quit after one round or continue.

## Stopping the game from the web console

Just as starting the game is triggered by OSGi, you can also stop the game through the OSGi framework. Go to the [web console](http://localhost:8080/system/console) and press the stop icon next to the game bundle. This will send a signal to the game thread to stop at a good point. If the game is still waiting for user input, then the game will wait for this user input first and then close itself. It is not considered good practice to kill the thread itself right away, so that's why this approach was chosen. After pressing stop the servicetracker is closed. Note that the buffered reader used in the game is not closed, because we should not close the `System.in` stream if we want to use it again later on.

After the bundle was stopped you can start it again in the web console. This should start the game again.