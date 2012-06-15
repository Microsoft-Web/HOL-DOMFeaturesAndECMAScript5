#DOM Features and ECMAScript5#

## Overview ##

Internet Explorer 10 contains lot of new Document Object Model (DOM) features and improvements to support ECMAScript 5, including advanced hit-testing APIs, media query listeners, and pointer and gesture events.
In this lab, we see how to use some of the new features, such as “**strict mode**”, which helps us avoid common errors in JavaScript by throwing errors while in the interpretation phase.
We also see how to use one of the advanced hit-testing APIs - **msElementsFromPoint** - to obtain the elements where we clicked.
Finally, we use the **createContextualFragment** method to pre-parse an object before it is inserting it into another object. In our case, we add an object into the HTML body element.

<a name="Objectives" />
### Objectives ###

In this hands on lab, you will learn how to:

- Use the ECMAScript 5 Strict Mode.
- Use new types, creating arrays to create filters.
- Catch click events using the new msElementsFromPoint method.
- Pre-parse objects, before appending them to the document, using the **createContextualFragment** function.

<a name="Prerequisites" />
### Prerequisites ###

- Internet Explorer 10

- [Visual Web Developer Express 2010](http://www.microsoft.com/web/gallery/install.aspx?appid=vwd)

- An HTML editor of your choice

- Prior knowledge of CSS

- Http context using IIS or IIS Express

<a name="Exercises" />
## Exercises ##

This hands on lab includes the following exercises:

1. [Exercise 1: Document Object Model (DOM)](#Exercise1)

1. [Exercise 2: JavaScript](#Exercise2)
 
Estimated time to complete this lab: **30-45 minutes**.

<a name="Exercise1" />
### Exercise 1: Document Object Model (DOM) ###

This exercise focuses on two of the several new Document Object Model (DOM) features.
**CSSOM View** introduces a method called **elementFromPoint**, which returns the element under an x- and y-coordinate in a viewport. This works well for single elements. For example, passing coordinates of an element, this method returns this element. 
Internet Explorer 10, uses JavaScript to introduces the **msElementsFromPoint** method, which returns a node list of all elements that intersect the point at the given x- and y-coordinates or a rectangular area, respectively.
The **createContextualFragment** method allows us to parse a string into a **DocumentFragment** object. Unlike inserting markup into innerHTML, the method create pre-parsed content to be added or inserted into a document when needed later.

<a name="Ex1Task1" />
#### Task 1 - Advanced hit-testing APIs ####

This lab uses the **msElementsFromPoint** method, to handle the click event on the assets. When a click event is received, we get a list of nodes that are under or intersect the x/y coordinates.

1.	Open the solution under the **Source\Begin** folder and examine the **TheFacePlace** project.
1. Now, open the **NewGame.TopMenu.js** file, located in the **Scripts** folder.
1. Add a call to a new function, which will handle the **click** event, when the user clicks over any asset.
 	
 	<!-- mark:7 -->
 	````CSS
	...
	var topMenu = {
	    init: function (im) {
	        imageHandler = im;
	
	        this.initAssets();
	        this.initIconsEvents();
	    },
	...
	````

1.	Implement this **initIconEvents.** In this function, you can see how the new querySelector API is used to select all the **assetsOverlay** class objects, to which we attach a handler for the click event. This event handler uses hit-testing to determine which items were clicked and then gets the name of the function to be called.

	<!-- mark:6-33 -->
	````CSS
	...
	initAssets: function () {
	    createAssets();
	},
	 
	initIconsEvents: function () {
	    var objTarget, func, param;
	 
	    $('.assets > img') 
	        .mouseenter(function () {
	            $(this).addClass('zoomInImg');
	        })
	        .mouseleave(function () {
	            $(this).removeClass('zoomInImg');
	        });
	 
	    if (document.msElementsFromPoint) {
	        document.querySelector(".assetsOverlay").addEventListener("click", function (e) {
	            var targets = document.msElementsFromPoint(e.clientX, e.clientY);
	 
	            if (targets.length > 0 && targets[0] !== undefined) {
	                objTarget = targets[0];
	 
	                func = objTarget.getAttribute("data-function");
	                param = objTarget.getAttribute("data-params");
	 
	                if (func !== undefined && NewGame.TopMenu[func] !== undefined && param !== undefined && param !== "") {
	                    NewGame.TopMenu[func].call(window, param);
	                }
	            }
	        }, false);
	    }
	},
	...
	````

	> **Note:** If you want to know more about **msElementsFromPoint** API, refer to [http://go.microsoft.com/fwlink/p/?LinkId=233342](http://go.microsoft.com/fwlink/p/?LinkId=233342)

<a name="Ex1Task2" />
#### Task 2 - Creating the Asset’s Tool Tip ####


In this task, we create the asset’s tool tip using the **createContextualFragment** method to pre-parse the HTML string before appending it to the document.

1.	Open the **NewGame.TopMenu.js** file, if not already opened from the previous task.
1.	Find the **init** function implementation and add a new call to **initToolTips**
 	
 	<!-- mark:6 -->
 	````CSS
	...
	init: function (im) {
	    imageHandler = im;
	        this.initAssets();
	        this.initIconsEvents();
	        this.initToolTips();
	},
	...
	````

1.	Add the **initToolTips** implementation. The first line of this function calls the **createToolTip** function, creating an object to be inserted into the **document.body** later. The function also subscribes to the “mouseenter” and “mouseleave” events, displaying and hiding tooltips, respectively.
	
	<!-- mark:5-18 -->
	````CSS
	...
	initIconsEvents: function () {
	...
	},
	initToolTips: function () {
		 var toolTipContextualFragment = createToolTip();
	 
		 if (toolTipContextualFragment !== null) {
			  $(".assets > img")
					.mouseenter(function (e) {
						 showTooltip(e, toolTipContextualFragment);
					})
					.mouseleave(function () {
						 hideTooltip();
					});
	 
		 }
	},
	...
	````

1.	Create a new function named **createToolTip.** This function creates and returns a **contextualFragment** object, which parses the string HTML passed before it’s added to the DOM.
	
	<!-- mark:5-28 -->
	````CSS
	...
	function createAssets () {
	}
	
	function createToolTip() {
		 var htmlTxt, rangeObj, contextualFragment;
	 
		 if (document.createRange) {
			  rangeObj = document.createRange();
			  if (rangeObj.createContextualFragment) {
					htmlTxt = "<div class=\"toolTip\"></div>";
					
					try {
						 contextualFragment = rangeObj.createContextualFragment(htmlTxt); 
	 
						 return contextualFragment; 
					}
					catch (e) { 
						 // Some browsers don't support this method correctly, 
						 // so an error is being thrown.
					}
	
					return null;
			  }
		 }
	 
		 return null;
	}
	...
	````

1.	Implement the **showToolTip** and **hideToolTip** functions. The **showToolTip** function inserts the object contained in the **ContextualFragment** into the **document.body**. After this object is appended to the body object, **ContextualFragment** releases this object and remains empty.
	
	<!-- mark:6-29 -->
	````CSS
	...
	function createToolTip() {
	...
	}
	 
	function showTooltip(e, toolTipContextualFragment) {
		 //Insert the tool tip to body if ContextualFragment has an object
		 if (toolTipContextualFragment.childNodes.length === 1) {
			  document.body.insertBefore(toolTipContextualFragment, document.body.firstChild);
		 }
	 
		 var x, y, toolTip = $('.toolTip');
	 
		 if (toolTip.length) {
			  toolTip.html($(e.target).attr('data-toolTipText'));
	 
			  x = $(e.currentTarget).offset().left - (toolTip.width() - e.currentTarget.clientWidth) / 2;
			  y = $('.assetsWrapper').position().top + $('.assetsWrapper').height() - (toolTip.height() / 2);
	 
			  toolTip.css("left", x);
			  toolTip.css("top", y);
	 
			  toolTip.show();
		 }
	}
	 
	function hideTooltip() {
		 $('.toolTip').hide();
	}

	window.NewGame.TopMenu = topMenu;
	````

This task concludes part one of the lab.

<a name="Exercise2" />
### Exercise 2: JavaScript ###

In this exercise, we learn both how to use **“strict mode”** and its importance. Strict mode was introduced in ECMAScript5 to protect the developer from the more perilous aspects of the language.

Do you want to learn more? Follow the link: <http://ecma262-5.com/ELS5_HTML.htm#Annex_C>

We also learn about **Typed Arrays**, and see an implementation of one of them. Typed Arrays are a hash lookup, which you can access very quickly, using JavaScript. Typed arrays can also be used to manage in-memory binary data. A number of types are supported, such as Int8Array, Int16Array, Int23Array, their respective unsigned integer counterparts, Float32Array and Float64Array.

Learn more about “Type Arrays” at:  <http://msdn.microsoft.com/en-us/library/br212485(v=vs.94).aspx>

<a name="Ex2Task1" />
#### Task 1 - Using ECMAScript 5 “strict mode” ####

This lab demonstrates the importance of using "ECMAScript 5 strict mode".  ECMAScript 5 strict mode is intended to improve error checking and identify script that might not be resilient to future versions of JavaScript. “strict mode” throws errors, which would otherwise be “silent fails”. If you want to see the complete table of errors and extend your knowledge, please refer to <http://msdn.microsoft.com/en-us/library/br230269(v=vs.94).aspx>.

1.	If not already open, navigate to the **Source\Begin** folder and open the **TheFacePlace** project.

1.	Before starting, configure IE10 to show every JavaScript error. To do this, open the “Internet Options” popup and click the “Advanced” tab. Uncheck both checkboxes for “Disable script debugging”. This displays JavaScript errors so we can see the problems we have when we use “strict mode”.

	![uncheck-both-disable-script-debugging-checkboxes](images/uncheck-both-disable-script-debugging-checkbo.png?raw=true)

	_Uncheck both “Disable script debugging” checkboxes_

1.	Start the application and navigate to the **NewGame** page by clicking in the **Start** button, located at the bottom of the page. Verify that no errors appear.

	> **Note:** You can use the **isDebug=1** parameter to pre-load an image. URL sample: <http://localhost:{port}/NewGame.htm?IsDebug=1>

1.	Now that you are sure we have a functional game, open the **NewGame.TopMenu.js** file, located in the Scripts folder and add “**use strict**” at the beginning of the first anonymous function. “use strict” can be added also at the beginning of the file or at the beginning of every block. This kind of declaration is known as a _directive prologue_.

	<!-- mark:2 -->
	````CSS
	(function () {
		 "use strict";
		 imageHandler = null;
		 var isAppliedFilter = false;
		 var assetsJson = {
			  Eyes: {
	...
	````

1. Refresh the same page. You should get a JavaScript error reported by the browser.

	
	![javascript-error-popup](images/javascript-error-popup.png?raw=true)

	_JavaScript Error Popup_

1.	This error occurs because the browser cannot finish the interpretation of the file, so the entire anonymous function is ignored. Click **No** to close the error dialog.
1.	To avoid this error, fix all JavaScript errors. Localize the **imageHandler** declaration by adding the **var** keyword.
	
	<!-- mark:3 -->
	````CSS
	(function () {
		 "use strict";
		 var imageHandler = null;
		 var isAppliedFilter = false;
		 var assetsJson = {
			  Eyes: {
	...
	````

1.	Refresh the page. You should still get a different error. To fix it, remove one of the duplicated **ImagesPath** properties in the **Eyes** object.


	<!-- strike:4 -->
	````CSS
	...
	var assetsJson = {
		 Eyes: {
			  ImagesPath: 'Images/FaceAssets/Eyes/',
			  ImagesPath: 'Images/FaceAssets/Eyes/',
			  DataFunction: 'addEyes',
			  CssClass: 'eyes',
			  Icons: [
	...
	````

1. Find the **addEyes** function declaration and remove the duplicated **assets** parameter

	````CSS
	...
	addEyes: function (assets, assets) {
		 var asset = splitAndTrimAsset(assets, 2);
	 
		 if (asset !== null) {
	...
	````

1.	After all errors are fixed, refresh the page and everything should run correctly.
1.	Having learnt the importance of using “strict mode”, open the remaining JavaScript files related to the game (except the 3rd party libraries such as jQuery or KineticJS) and add “strict mode”. 

	The files list is:
	- Face.Database.js
	- Face.DragAndDropManager.js
	- Face.ImageManager.js
	- Face.js
	- Face.Pager.js
	- ImageHandler.js
	- ImageHandler.Kinetic.js
	- ImageHandler.Kinetic.Undo.js
	- InstructionsPageScript.js
	- NewGame.BackgroundFileManager.js
	- NewGame.js
	- NewGame.TopMenu.js
	- OpenGame.js
	- ShadowPopup.js

1. Test the application to ensure everything runs correctly.

<a name="Ex2Task2" />
#### Task 2 - Creating the Brightness Filter ####

In this task we use **Typed Arrays** to implement the filters we can apply to the background. In this case we see how to implement two of the filters: brighter and darker filters. 
The filters also use Web Workers, to improve the performance of the game. More information about Web Workers can be found in the New HTML5 Features lab.
	
1.	Open the **ImageHandler.Filters.js** file.
1.	Localize the switch statement at the top of the file and add two more ‘cases’, which will perform the **brightness** and **darkness** filters.
	
	<!-- mark:15-20 -->
	````CSS
	...
	self.addEventListener('message', function (e) {
		 var data;
	 
		 switch (e.data.effect) {
			  case 'grayscale':
					data = applyFilter(e.data.imageData, grayscale);
					break;
			  case 'invert':
					data = applyFilter(e.data.imageData, invert);
					break;
			  case 'luminance':
					data = applyFilter(e.data.imageData, luminance);
					break;
			  case 'brighter':
					data = brightnessContrast(e.data.imageData, 0.25, 1.5);
					break;
			  case 'darker':
					data = brightnessContrast(e.data.imageData, -0.25, 1.5);
					break;
	...
	````

1.	Create a new function called **brightnessContrast**, which applies these filters according to the **brightness** and **contrast** parameters set.

	<!-- mark:5-14 -->
	````CSS
	...	
	function luminance(d, i) {
	}

	function brightnessContrast(imageData, brightness, contrast) {
		 var lut = brightnessContrastLUT(brightness, contrast);
		 var lutJson = { r: lut, g: lut, b: lut, a: identityLUT() };
	 
		 return applyFilter(imageData, function (d, i) {
			  d[i] = lutJson.r[d[i]];
			  d[i + 1] = lutJson.g[d[i + 1]];
			  d[i + 2] = lutJson.b[d[i + 2]];
			  d[i + 3] = lutJson.a[d[i + 3]];
		 });
	}
	...
	````

1.	In this function, we call another function named **brightnessContrastLUT**, which creates a uint8Array, and returns this array, having previously set each array row, calculating each row based on the **brightness** and **contrast** parameters. 

	<!-- mark:2-12 -->
	````CSS
	...
	function brightnessContrastLUT(brightness, contrast) {
		 var lut = getUint8Array(256);
		 var contrastAdjust = -128 * contrast + 128;
		 var brightnessAdjust = 255 * brightness;
		 var adjust = contrastAdjust + brightnessAdjust;
		 for (var i = 0; i < lut.length; i++) {
			  var c = i * contrast + adjust;
			  lut[i] = c < 0 ? 0 : (c > 255 ? 255 : c);
		 }
		 return lut;
	}
	...
	````

1.	The previous function uses an **8-bit unsigned integer array**, created by using a JavaScript function called **Uint8Array**.

	<!-- mark:2-4 -->
	````CSS
	...
	function getUint8Array(len) {
		 return new Uint8Array(len);
	}
	...
	````

	> **Note:** If you want to know more about new JavaScript types supported by IE10, refer to <http://msdn.microsoft.com/en-us/library/ie/br212485(v=vs.85).aspx> or if you want to expand your knowledge about JavaScript types, refer to <http://go.microsoft.com/fwlink/p/?LinkID=245657>

1.	The final task is to implement another function **identityLUT**, which returns a very simple array, according to the index of each row in the array.

	<!-- mark:2-8 -->
	````CSS
	...
	function identityLUT() {
		 var lut = getUint8Array(256);
		 for (var i = 0; i < lut.length; i++) {
			  lut[i] = i;
		 }
		 return lut;
	}
	...
	````



1.	Open the game, add a background and apply the **brightness** or **darkness** filter. If you use the image for testing, passing “**isDebug=1”** parameter, you should see an image as displayed below (if you apply the brightness filter).

	![applying-the-brightness-filter](images/applying-the-brightness-filter.png?raw=true)
	
	_Applying the brightness filter_

This task concludes this exercise and the whole lab.

<a name="Summary" />
## Summary ##

We saw the importance of using “**ECMAScript 5 strict mode**”, to avoid semantics problems. In addition, we use one of the **Typed arrays** to make a filter which can be applied to the game background.
Regarding **Document Object Model (DOM) features**, we used the **msElementsFromPoint** method to detect where the click was fired, and we used the **createContextualFragment** method to create a pre-parsed object before inserting it into the document.
