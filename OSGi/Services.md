# Services in OSGi

OSGi offers a Services registry where services can be found, registered and unregistered. This registry functionality is not present in the bare Java platform. 

We'll explore the advantages of Services by making an example project.

## Creating the API
Creating an interface (or API) is a good software engineering practice of decoupling the specific implementation from the generic functionality. The interface serves as the agreement between the calling code and the implementing code. The caller sees only the interface, and in this way the implementing code may be optimized or improved or replaced as long as the interface is adhered to.

In this project we will make a service that randomly mixes the letters of words: the garbler.
We start off by making an interface that merely states that any implementer should have the method `garbleWord` accepting a String and returning a String.

Start off by starting a new workspace and adding the special `cnf` folder. This was described in the previous chapter.

The next step is to create a new bundle for our API called `org.flexiblepower.tutorial.api` (`File` -> `New Bndtools OSGi project`).
Eclipse may give an error regarding the `res` folder. This is solved by right-clicking the `org.flexiblepower.tutorial.api` project and adding a new folder called `res` to the root of the project. Also the template may have added `osgi.annotations` to the build path (look for it in the `build` tab of the `bnd.bnd` file), so let's remove this unused bundle. Let's add the `osgi.core` bundle as well for our service to be ready to use the OSGi functionality we'll be needing.

Now we'll add the interface definition to this project. Add a new interface to the `src` folder called GarbleService and define the interface using one public method that takes a String returns a String and is called `garbleWord`.

## Creating a service

Now that we've put in place the api, let's make an actual implementation. We'll to add the implementation to a separate bundle to nicely separate the interface from the implementation. Let's call the new bundle `org.flexiblepower.tutorial.services` and set it up in the same way as the api project (as described in the previous paragraph).

Now we'll add the API bundle to the build path (using the `build` tab of the `bnd.bnd` file of the `services` bundle). Only after we've done this can we reference the API from code.



**Comments
Using the `BundleContext`

Demo with creating an API (e.g. the `Printer`) and a service provider (e.g. `SysOutPrinter`).

Show the service repository in the webconsole
**


## Finding a service

Using the `ServiceTracker`

Demo with creating two service consumers (e.g. `HelloSayer` and `BonjourSayer`).

## Registration properties

Demo by adding a type property for the service provider and creating a second implementation (e.g. the `SysErrPrinter`)

## Filtering on properties

Demo by limiting 1 of the service consumers to a single type of `Printer`

Also show that there is no direct dependency from the consumer bundle to the provider bundle, only on the API bundle.
