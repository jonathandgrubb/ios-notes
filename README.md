# ios-notes
Useful notes about iOS programming and the Swift programming language

Welcome to the ios-notes wiki!

### guard (keyword):
use it like a reverse assert. appears to be like unwrapping, but it's for validating the presence of required stuff

### REST:

    let methodArguments : [String : AnyObject] = [
        "method": "flickr.photos.search",
        "api_key": API_KEY,
        "extras": EXTRAS,
        "format": DATA_FORMAT,
        "nojsoncallback": NO_JSON_CALLBACK,
        "text": phraseSearchText.text!
    ]
        
    /* 3 - Initialize session and url */
    let session = NSURLSession.sharedSession()
    let urlString = BASE_URL + escapedParameters(methodArguments)
    let url = NSURL(string: urlString)!
    let request = NSURLRequest(URL: url)
    
    /* 4 - Initialize task for getting data */
    let task = session.dataTaskWithRequest(request) { (data, response, error) in
        
        /* 5 - Check for a successful response */
        /* GUARD: Was there an error? */
    
    }

    /* 9 - start the request */
    task.resume()


### JSON parsing:

        /* 6 - Parse the data (i.e. convert the data to JSON and look for values!) */
        let parsedResult: AnyObject!
        do {
            parsedResult = try NSJSONSerialization.JSONObjectWithData(data, options: .AllowFragments)
        } catch {
            parsedResult = nil
            print("Could not parse the data as JSON: '\(data)'")
            return
        }
            
        /* GUARD: Did Flickr return an error (stat != ok)? */
        guard let stat = parsedResult["stat"] as? String where stat == "ok" else {
            print("Flickr API returned an error. See error code and message in \(parsedResult)")
            return
        }
            
        /* GUARD: Are the "photos" and "photo" keys in our result? */
        guard let photosDictionary = parsedResult["photos"] as? NSDictionary,
            photoArray = photosDictionary["photo"] as? [[String: AnyObject]] else {
                print("Cannot find keys 'photos' and 'photo' in \(parsedResult)")
                return
        }
            
        var totalImages: Int = 0
        for photo in photoArray {
            if photo["url_m"] != nil {
                totalImages++
            }
        }

### JSON types to Swift Types:

	null ==> nil
	Number ==> Int, Float, Double, ...
	String ==> String
	Boolean ==> Bool
	Array ==> [AnyObject]
	Object ==> [String:AnyObject]
	Array of objects ==> [[String:AnyObject]]

### Basic GET:

    overide func viewDidLoad() {
        super.viewDidLoad()

        let imageURL = NSURL(string: "https://www.mydogtrainingonline.com/images/how-to-train-your-dog.jpg")

        let task = NSURLSession.sharedSession().dataTaskWithUrl(imageURL) { (data, response, error) in

            if error == nil {
                let downloadedImage = UIImage(data: data!)

                performUpdatesOnMain {
                    self.imageView.image = downloadedImage
                }
            }

        }

        task.resume()
    }

* note - security will not allow you to use this method with HTTP unless you change something in info.plist.info

        (Generic ALLOW HTTP FOR EVERYTHING)
        > App Transport Security Settings
            > Allow Arbitrary Loads = YES
              (this can also be set per domain somehow)

        (More specific per domain/subdomain)
        > App Transport Security Settings
            > Exception Domains
                > themoviedb.org
                    > NSExceptionAllowsInsecureHTTPLoads = YES
                    > NSIncludesSubdomains = YES        

### Apple's way to build urls: 
NSURLComponents, NSURLQueryItem
(ensures that it only contains safe ASCII characters)

    let components = NSURLComponents
    components.scheme = "https"
    components.host = "api.flickr.com"
    components.path = "/services/rest"
    components.queryItems = [NSURLQueryItem]()

    let queryItem1 = NSURLqueryItem(name: "method", value: "flickr.photos.search")
    let queryItem2 = NSURLqueryItem(name: "api_kehy", value: "1234")
    let queryItem3 = NSURLqueryItem(name: "text", value: "purple")
    
    components.queryItems!.append(queryItem1)
    components.queryItems!.append(queryItem2)
    components.queryItems!.append(queryItem3)

    print(components.URL!)

How they did it in the class:
    
    private func flickrURLFromParameters(parameters: [String:AnyObject]) -> NSURL {
        
        let components = NSURLComponents()
        components.scheme = Constants.Flickr.APIScheme
        components.host = Constants.Flickr.APIHost
        components.path = Constants.Flickr.APIPath
        components.queryItems = [NSURLQueryItem]()
        
        for (key, value) in parameters {
            let queryItem = NSURLQueryItem(name: key, value: "\(value)")
            components.queryItems!.append(queryItem)
        }
        
        return components.URL!
    }


### Chaining network requests:

    func request1() {
        /* initialize session and url */
        let session = NSURLSession.sharedSession()
        let urlString = BASE_URL + escapedParameters(methodArguments)
        let url = NSURL(string: urlString)!
        let request = NSURLRequest(URL: url)
        
        /* Initialize task for getting data */
        let task = session.dataTaskWithRequest(request) { (data, response, error) in
            
            // parse response 1
            response1data = ...

            // call request 2 (with some output of response 1)
            request2(response1data)
        }

        // start the request
        task.resume()
    }

    func request2(requestData) {
        /* initialize session and url */
        let session = NSURLSession.sharedSession()
        let urlString = BASE_URL + escapedParameters(methodArguments)
        let url = NSURL(string: urlString)!
        let request = NSURLRequest(URL: url)
        
        /* Initialize task for getting data */
        let task = session.dataTaskWithRequest(request) { (data, response, error) in
            
            // parse response 2
            response2data = ...

            // reenaable the UI
            performUIUpdatesOnMain {
                self.setUIEnabled(true)
            }
        }

        // start the request
        task.resume()
    }

    func performUIUpdatesOnMain(updates: () -> Void) {
        dispatch_async(dispatch_get_main_queue()) {
            updates()
        }
    }

### Authentication:
Authentication — ensuring someone’s identity (i.e. “we’ve confirmed this user is indeed Jarrod Parkes”)
Authorization — providing someone access to something (i.e. “this user is allowed to post ratings for movies”)
User Data vs. Anonymous Data
Login with API vs. Login on web site


### Completion Handlers:
Completion handlers are closures for a calling method to do stuff with the results of a method it has called... they are used ALL THE TIME

    func usefulMethod(arg1: String, completionHandlerForUsefulMethod: (result: String, error: String?) -> Void) -> Bool {
        // do some stuff
        if arg1 == "do something useful" {
            completionHandlerForUsefulMethod("success!!!", nil)
        } else {
            completionHandlerForUsefulMethod(nil, "Big problems!!!")
        }
    }

    func callingMethod {
        usefulMethod("do something useful") { (result, error) in
            if let error = error {
                print "Awww man!!!"
            } else {
                print result
            }

        }
    }

### Closures

Are first class citizens of Swift just like a regular type.
They can be assigned to a variable, array, dictionary, etc.

	let f = {(x:Int) -> Int
	        in
	        return x + 42}

	f(0)
	f(1)
	f(99)

You can assign closures / functions of the same type to the same array. Note that all of the members of the array must match the signature of the first element: (Int)->Int

	let closures = [f,                      // our previous closure
	    {(x:Int) -> Int in return x * 2},   // a new Int -> Int closure
	    {x in return x - 8},                // no need for the type of the closure!
	    {x in x * x},                       // no need for return if only one line
	    {$0 * 42}                           // access parameter by position instead of name

	// can even iterate through it

	for fn in closures{
	    fn(42)
	}

Functions and closures are exactly the same thing

	func foo(x:Int)->Int{
	    return 42 + x
	}

	let bar = {(x: Int)->Int in
	    return 42 + x
	}

List (not exhaustive) of ways to declare closures:
http://goshdarnclosuresyntax.com/

### typealias

you can make an alias of an existing type into a new type (like typedef in C)

	// old type to new type
	typealias <new_type> = <existing_type>
	typealias Integer = Int

	// can also do it to a function (closure) signature
	typealias IntToInt = (Int)->Int
	typealias IntMaker = (Void)->Int


## Variable Capture

With what we know about closures we can capture variables

	func makeCounter() ->IntMaker{
    	var n = 0
    
    	// Functions within functions?
    	// Yes we can!
    	func adder()->Integer{
        	n = n + 1
        	return n 
    	}
    	return adder
	}

	let counter1 = makeCounter()
	let counter2 = makeCounter()

	// will increment its own copy of the variable 'n'
	counter1()
	counter1()
	counter1()

# GCD (Grand Central Dispatch)

GCD makes asynchronous (concurrent) programming easier by hiding threads from the developer. You provide closures to a queue and GCD will run them for you.

There is a synchronous queue (stuff is run in order) and an asynchronous (concurrent) queue (stuff isn't run in order) type

Each queue will run on its own thread and will not block each other.

GCD is written in C so the functions to access GCD resources look like C:

### Main Queue
There is a main queue (don't block this). Most of the time the network will block this and the UI will not be able to update, but other things computationally expensive can cause this too (large images, applying filters to a video, etc.) 

Get the main queue (for the UI)

	dispatch_get_main_queue()

### Global Queues (4 queues)

The global queues:

	dispatch_get_global_queue()

	QOS_CLASS_USER_INTERACTIVE		// top priority
    QOS_CLASS_USER_INITIATED		// regular priority
    QOS_CLASS_BACKGROUND			// low priority
    QOS_CLASS_UTILITY				// lowest priority

### Putting closures on the queue

	dispatch_async()  // accepts a queue and a closure

### Creating queues

represents a queue

	dispatch_queue_t
	
creates a serial queue, but we won't b/c iOS provides a bunch already

	dispatch_queue_create("new_queue_name", DISPATCH_QUEUE_SERIAL)
	dispatch_queue_create("new_queue_name", nil)

creates a concurrent queue

	dispatch_queue_create("new_queue_name", DISPATCH_QUEUE_CONCURRENT)

### Another example:

	// put something on the highest priority global queue
	// this will print 'tac' first and then print 'tic' in the very near future
	let q = dispatch_get_global_queue(QOS_CLASS_USER_INTERRACTIVE, 0)

	dispatch_async(q) { () -> Void in
	    print("tic")
	}
	print("tac")

### Thread Safety

When a framework can run in the background it is said to be "thread-safe".  Frameworks that are not thread-safe and can only run in the main queue:

* core data
* uikit 
  * although some parts of this are thread safe. ex. UIKit
  * "if it ends in View, it belongs on the main queue"

It's a good idea to make sure all of your completion handlers run in the main queue, since a completion handler is nearly always updating the UI.


### Persistence and Core Data

#### NSUserDefaults (PickYourPitch app example)
* can store stuff in a plist
* following datatypes: Data, String, Number, Array, Dictionary
* can only store < 1MB
* there is a db that stores:
  * system-wide defaults
  * language defaults
  * app-specific defaults
  * more

Accessing the data (which is a dictionary)

    NSUserDefaults.standardUserDefaults().valueForKey(key: String)
                                         .stringForKey(key: String)
                                         .boolForKey(key: String)
                                         .floatForKey(key: String)
                                         etc.

Setting the data

    NSUserDefaults.standardUserDefaults().setValue(value: AnyObject?, forKey: String)

Resetting the data
* Simulator ~ Reset Content and Settings
  * to make it look like this the the first time we have ever run "this" app
  * clears out the app-specific data

In the AppDelegate put a call to something like the first function (below) within the second function to initialize the app-specific defaults so you don't clutter up the viewDidLoad in the controller

    checkIfFirstLaunch() 

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool 

#### Where is the data stored?

#### What is the sandbox? (SandboxPlayground app example)
* where the app keeps all of its stuff and is private from other apps
* iOS has Unix-based file system where is app is its own island
* has a Documents and Library folder for each app
* can sync using iCloud
* absolute paths for folders will change

has containers:
* Bundle Container: executable code and resources
* Data Container: all of the user and app data
  * "Documents/" - (user data) - iCloud will back this up / important stuff
  * "Library/" - (non user data) - files you don't want to expose to the user 
    * (this is where NSUserDefault is stored: "Library/Preferences/info.myapp.mobile.plist")
  * "Temp/" & "Caches/" - iCloud will never back these up and they are purged periodically
* iCloud Container

Sample reading and writing to a file in the sandbox:

    func sandboxPlayground() {
        let fm = NSFileManager.defaultManager()
        let urls = fm.URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask)
        
        let url = urls.last?.URLByAppendingPathComponent("file.txt")
        
        do {
            try "Hi there!".writeToURL(url!, atomically: true, encoding: NSUTF8StringEncoding)
        } catch {
            print("Error while writing")
        }
        
        do {
            let content = try String(contentsOfURL: url!, encoding: NSUTF8StringEncoding)
            
            if (content == "Hi there!") {
                print("yay!")
            } else {
                print("oops")
            }
        } catch {
            print("Something went wrong")
        }
    }


#### CORE Data (CoolNotes app example)

Model Layer (Core Data Stack):

* Object Model   - (a.k.a. Managed Object Model) specifies app's classes and relationships (this is the Model file you create with UI)
* Managed Object - classes that save stuff to the DB file (subclass of NSManagedObject) example: character in a game
* Context        - where objects live and operations take place (NSManagedObjectContext)
* Fetch Request  - what's living inside the Context? performs searches or 'fetch' requests
* Stores         - where Managed Objects are stored to disk (XML, binary, SQLite)
* Store Coordinator - allows you to have multiple stores

Controller Layer:

Fetched Results Controller - controls how data from a fetch is displayed in a view (TableView/CollectionView) (NSFetchResultsController)

Model file you create an sql-like database that you can query. The UI contains

* Entities: "Tables"
* Attributes: "Columns"
* Relationships: between the Entities (1-to-1, 1-to-many)
* Indexes: index the things you know yoiu're going to be searching on to make the searches faster
* Deletion Rules: Nully (just get rid of the record), Cascade (get rid of this record and the records in this Relationship)

To go from the Model file to classes:

    Editor ~ Create NSManagedObject Subclass...

Two files will be created for each entity: a class file and an extention (this file is autogenerated... don't put new code here)

Example class file (defined by developer):

        class Notebook: NSManagedObject {

            // so we can create usable instances of ourself
            convenience init(name:String, context:NSManagedObjectContext) {
                
                // create an NSEntityDescription for "Notebook"
                if let ent = NSEntityDescription.entityForName("Notebook", inManagedObjectContext: context) {
                    // create a new instance
                    self.init(entity: ent, insertIntoManagedObjectContext: context)
                    // set the properties
                    self.name = name
                    self.creationDate = NSDate()
                } else {
                    // couldn't create an NSEntityDescription for "Note"
                    fatalError("Unable to find Entity Name!")
                }
            }
            
            // offer a better way to view the creationDate in the app
            var humanReadableAge : String {
                get {
                    let fmt = NSDateFormatter()
                    fmt.timeStyle = .NoStyle
                    fmt.dateStyle = .ShortStyle
                    fmt.doesRelativeDateFormatting = true
                    fmt.locale = NSLocale.currentLocale()
                    
                    return fmt.stringFromDate(creationDate!)
                }
            }
        }

NSPredicate - to filter members of an array with a very simple language

        let p = NSPredicate(format: "notebook = %@", argumentArray: [argumentsToSubstituteIn])

Migrations

        Editor ~ Add Model Version
        File Inspector for Model ~ Current Version ~ <new version>

        CoreDataStack.swift
        // options for migration
        let options = [NSInferMappingModelAutomaticallyOption : true, NSMigratePersistantStoreAutomaticallyOption : true]

        do {
            try addStoreCoordinator(NSSQLiteStoreType, configuration: nil, storeURL: dbURL, optionals: options)
        } catch {
            print("unable to add store at \(dbURL)")
        }

Concurrency (CoolNotes Lesson6)

CORE Data actually is thread safe. The queue/thread that the context is created on is the only queue/thread is can be used on, though. If you have a lot of data to load in the background, do it in a background thread. The problem is communicating these changes to the main queue that is able to change the UI. There is now a way to notify the main queue when the parent/child relationship. 

child (background queue - get data) -> parent (main queue - update UI and Save to model)

If a lot of data has been loaded in the background (via presumably a web api) we will also want to save it to the model. This can be time intensive, so we might want to do the saving on yet another queue and make this a parent of the parent.

child (background queue - get data) -> parent (main queue - update UI) -> persist (background queue - Save to model)

See the sample project for details!
