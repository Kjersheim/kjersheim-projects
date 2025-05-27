---
title: "Full Stack Project: Recipe Web App Using Node, EJS, and MongoDB"
date: 2023-12-12
draft: false
summary: "A detailed walkthrough of a personal full stack project: building a dynamic recipe manager using Node.js, Express, EJS templating, and MongoDB."
---

# Project overview

[Link to repository](https://github.com/Kjersheim/recipe-webapp)

Full Stack Optional Project
Creating a recipe-webapp using Node, Mongoose, EJS and MongoDB.

```
Contents
1	Crafting a Solid Foundation	
2	Prototyping the Project	
2.1	Paper Sketches	
3	Working on the Design and Starting to Structure Files	
3.1	The Folder Structure	
3.2	Front-End Folder Structure and Files	
3.3	Back-End Folder Structure and Files	
4	Project Timeline	
5	Database Structure	
6	Web App Functionality	
6.1	Interface	
7	REST API Endpoints	
7.1	GET / and GET /about	
7.2	GET /browse	
7.3	GET /chosenrecipe/:id	
7.4	GET /create	
7.5	POST /create	
8	Helper Functions	
9	Strengths, Weaknesses, Final Reflections	
```





# 1	Crafting a Solid Foundation
As part of my Full Stack Programming course, I embarked on a journey into JavaScript, a lan-guage I had not previously explored. While I possessed foundational knowledge of HTML and CSS from prior courses, my experience had been limited to personal web development projects. However, the early course assignments sparked my interest in JavaScript, prompting me to en-roll in a dedicated JavaScript course.

This learning venture involved resources like javascript.info and Scrimba, where I honed my skills by creating various small applications, including calculators and simple games. Scrimba introduced me to EJS (Embedded JavaScript), a fusion with HTML that intrigued me and later found its place in my project. For this project in particular, a great source of help was gained from using info from Raddy's food recipe project [which can be found here.](https://github.com/RaddyTheBrand/RecipeBlog-MongoDB-Node.js)

My experiences with the Scrimba course included an examination of JavaScript techniques, some of which required substantial effort to grasp fully. Leveraging online resources, such as tutorials and guides, became a valuable aspect of my learning journey.
The genesis of this project emerged from a personal desire—a desire to fill an empty green rec-ipe book in my kitchen. Despite my love for cooking and recipe preservation, the book re-mained blank. Creating a web application to store and access recipes became an appealing so-lution, with MongoDB chosen as the database. While the initial vision included image upload-ing, practical considerations led to a more streamlined approach. The primary goal was to es-tablish a robust foundation for the project, emphasizing structured folder organization and adherence to best practices.
This documentation aims to provide an in-depth understanding of the project's architecture, components, and functionality. It chronicles the development journey and offers insights into its inner workings.

# 2	Prototyping the Project
The project journey commenced with a pragmatic blend of paper and digital prototyping tech-niques. As a student project, it began with a focus on exploring ideas and formulating a com-prehensive plan. The initial phase involved crafting a paper prototype, serving as a rudimentary yet effective tool for outlining the fundamental features of the project. This manual approach facilitated swift ideation without the complexities of intricate digital tools.

While contemplating the shift towards a more comprehensive digital prototype using tools like Figma, which we had employed in previous courses, or InVision, a choice for some of my personal projects, the development took a different path. Instead, substantial time was devot-ed to refining the HTML and CSS design. The goal was to create a visually appealing and func-tional user interface that resonated with the project's core objectives. The pursuit of a clean and user-friendly aesthetic presented its challenges, yet the project's development maintained its course.

Despite not being fully content with the visual aspects of the project and facing time con-straints, the design remained suitable, adaptable for both larger and smaller user interfaces. The evolving framework, coupled with the organization of project files and directories, trans-formed into a well-defined vision that continued to be iteratively developed. While recognizing the merits of employing advanced prototyping tools, the project's essence lay in its role as a personal endeavor, reflecting the journey of a student developer navigating the intricacies of full-stack development.

## 2.1	Paper Sketches
The initial phase of design and prototyping commenced with a manual approach involving pencil sketches on paper. The primary objective was to outline the fundamental structural components of the web page, with a particular emphasis on responsive design catering to both small mobile resolutions and larger desktop screens. These sketches served as the foundational framework upon which the project's visual elements would be built (Figure 1).

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/a6c5f269-e6eb-442c-afc6-64f279bc20e1)

During the paper-based prototyping phase, careful consideration was given to the precise placement of essential elements, including the navigation system and primary content sections. Furthermore, functional aspects, such as the integration of a mobile-resolution hamburger menu, were investigated. Leveraging insights from past experiences, the decision was made to adopt the hamburger menu design pattern, a successful strategy employed in prior web devel-opment projects (Figure 2).

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/e77eee71-567c-46e5-b925-75dd062df630)

To refine the design further, miniaturized representations of individual elements were crafted, and a canvas was employed to experiment with different layout arrangements (Figure 3 & 4). This iterative approach allowed for the exploration of diverse design concepts and their impact on the overall user experience. Concurrently, this phase ran in parallel with the development of HTML and CSS, significantly contributing to the evolution of the final design.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/cda47589-a0be-4605-8420-f44fb1486b2e)

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/17bc8d64-d39f-4202-b965-f88c6af21613)


# 3	Working on the Design and Starting to Structure Files
In tandem with the development of the design, various digital tools were used to enhance the visual elements of the project. Online resources such as https://app.haikei.app/ were employed for generating and testing shapes, http://www.icons8.com served as a valuable source for icons, and testing color palettes through tools like https://pigment.shapefactory.co/. These digital assets plays an important role in shaping the project's aesthetics and visual identity, as you have so many generated combinations and you can quickly see what would work – and what would maybe not work. As described above, instead of using digital tools I went into building the main HTML page and starting the CSS so I could work with these files while starting the pro-ject framework structure. With better time it would likely be very beneficial to use Figma or InVisio to create a functional and clean design. 

So then, simultaneously, a main focus was directed toward creating an organized folder struc-ture for the project. The primary objective was to adhere to best practices, drawing inspiration from exemplar projects that effectively segregated backend JavaScript and Node components into distinct directories. Furthermore, the client-side assets that were meant to be publicly ac-cessible were organized within a dedicated public folder. Discussions with fellow students and prior experience from publishing E08 Part 3 underscored the significance of creating a robust foundational structure from the project's inception. This early emphasis on structure has prov-en instrumental in maintaining a clear and efficient development process.

## 3.1	The Folder Structure
The project adopts a structured folder hierarchy to enhance organization, code maintainability, and efficient development. This hierarchical structure is thoughtfully designed to achieve spe-cific objectives, playing a big role in the overall organization of the project.
At the root level of the project, several essential files and directories are located. Among these, "app.js" serves as the entry point for the application and configures the Express.js server. The "package.json" file specifies project dependencies, scripts, and metadata, simplifying package management and automating installation procedures. For enhanced security, sensitive infor-mation like database connection strings is stored in the ".env" file, thus preventing such data from being included in version control.

The project's folder structure extends into directories like "public," "server," and "views." The "public" directory acts as a repository for publicly accessible assets, including stylesheets, client-side JavaScript, and images. This separation of assets from server-side code ensures efficient client-side delivery while adhering to security best practices. In contrast, the "server" directory houses server-related scripts and logic, including routes, controllers, and database interactions. This compartmentalization defines the server's functionality, promoting a clear separation of concerns between server and client components. Lastly, the "views" directory contains EJS (Em-bedded JavaScript) templates that define the application's front-end views. This structural seg-regation between server and client elements, facilitated by these specific folders, improves code maintainability, readability, and overall project organization.

The rationale behind structuring the folders in this manner is multifaceted. Firstly, it promotes modularity by segregating different aspects of the project into distinct directories, simplifying development by isolating and maintaining individual components. Clarity is another essential consideration, as an organized structure enhances code readability and comprehension. This structured approach allows developers to efficiently locate and work on specific parts of the application without confusion. 
Furthermore, this structure aligns with common web development conventions, facilitating col-laboration among team members. Security is also a priority, with sensitive data stored in the ".env" file, enhancing application security by preventing the exposure of critical information through version control. Additionally, the presence of a ".gitignore" file ensures that unneces-sary files are excluded from version control, reducing repository clutter and the risk of inad-vertent exposure of sensitive data.

In summary, the well-designed folder hierarchy establishes the groundwork for an organized and maintainable project. It streamlines development processes, paving the way for future en-hancements and effective collaboration.

## 3.2	Front-End Folder Structure and Files
As the project took shape, attention turned to organizing the front-end resources efficiently. The front-end folder structure was designed with the goal of maintaining a clear separation of concerns and promoting reusability. At the heart of this structure is the use of the EJS templat-ing engine, which played a big role in achieving modularity and consistency throughout the front-end components.

Views Folder: The "views" folder is central to the front-end architecture, housing the EJS tem-plates that define the application's views. Each EJS file represents a specific page or view within the application. This approach simplifies the process of creating new pages, as it allows for the reuse of the main layout while only altering the content within the <main> element. This mod-ularity enhances code maintainability and readability, ensuring that common elements like headers, footers, and navigation menus remain consistent across all pages.

Layouts Folder: Within the "views" folder, a "layouts" subfolder contains the primary layout template used throughout the application. This layout template includes the essential struc-ture of the HTML document, including the <head> section with metadata, CSS references, and the <body> element with navigation and the footer. The use of a consistent layout template simplifies the addition of new pages, ensuring a uniform look and feel across the application and easier modifications if that is needed.

Static Assets: Publicly accessible static assets, such as stylesheets, client-side JavaScript, and images, are stored in the "public" folder. This separation of assets from server-side code allows for efficient delivery to the client while adhering to security best practices. Stylesheets are or-ganized and categorized to maintain clean and maintainable CSS code, enhancing overall code quality.

EJS Files and Main Content: Within the "views" folder, EJS files define the content of each page. These files incorporate dynamic data, making use of EJS's templating capabilities to inject val-ues into the HTML markup. Additionally, EJS is employed for smaller changes, such as dynami-cally updating the title of each page to reflect its content or purpose.

In summary, the front-end folder structure and file organization in the project follow a modular and maintainable approach. The use of EJS templates for views, consistent layouts, and a clear separation of static assets contribute to a cohesive and user-friendly front-end architecture. This structure streamlines the addition of new pages and ensures a harmonious user experi-ence across the application.

## 3.3	Back-End Folder Structure and Files
The back-end folder structure and files play a vital role in organizing the server-side compo-nents of the project. These components are designed to handle data management, routing, and server functionality efficiently.

Server Folder: The "server" folder is the core of the back-end architecture, containing scripts and logic that drive the server's functionality. Key components within this directory include:

Routes: The "routes" subfolder houses route definitions that determine how the server re-sponds to various client requests. These routes handle tasks such as rendering views, pro-cessing form data, and interacting with the database.

Controllers: Within the "controllers" subfolder, controller scripts are responsible for processing incoming requests, interacting with the database, and orchestrating the flow of data between the server and client. This separation of concerns ensures a clean and organized codebase.

Helpers: Helper scripts, located within the "helpers" subfolder, are used to intercept and pro-cess incoming requests before they reach the route handlers. Middleware functions enhance the server's functionality by performing tasks like authentication, data validation, and error handling.

Database Interactions: The server-side code interacts with the MongoDB database through ded-icated scripts and modules. These interactions are encapsulated within the "models" subfolder, promoting modularity and maintainability.

The division of the back-end structure into specific directories and components serves multiple purposes. Firstly, it promotes modularity by separating different aspects of the server into dis-crete folders, simplifying development and maintenance. This separation of concerns enhances code readability, as I can quickly locate and work on specific parts of the server without confu-sion.

Additionally, this organized structure aligns with common web development conventions, facili-tating collaboration among team members, it that were to be the case in a different project. 


## 4	Project Timeline
Scheduling and structuring work in a project are critical components of successful project management. A well-defined timeline provides a clear roadmap for project development, ensuring that tasks are completed efficiently and in a systematic manner. It aids in tracking progress, meeting deadlines, and allocating resources effectively. In the context of this project, a structured timeline played a role in prioritizing and structuring the tasks within the various stages of ideation, design, development, and testing. The following table outlines the key milestones and activities during the project's phases.

|Dates|	Duration|	Focus|
|---|---|---|
|14- 15th Oct.|	5 hours|	Research and Ideation|
|20th Oct.	|3 hours	|Design Iterations and Logo Creation|
|21st Oct.	|4 hours|	Paper Models and Directory Setup|
|27-28th Oct.	|8-10 hours|	Directory setup, design testing, adding initial files app.js routes and controller file.|
|2nd Nov.	|2 hours|	Logo creation testing, AI and Canvas|
|6th Nov. 	|6-8 hours|	Working on app.js and routes for each html page, and design HTML/CSS testing. Setting up MongoDB and database js|
|7th Nov. 	|5 hours|	HTML/CSS, setting up GET controllers for all pages|
|11-12th Nov.	|10 hours|	HTML/CSS mainpage + create form page design.|
|13th Nov. 	|9-10 hours|	Setting up Recipe.js model, updating packages used and testing with create form |
|14th Nov. 	|8 hours|	Testing form design, adding helper functions. Updating GET methods. |
|20th Nov. 	|9 hours|	Testing Create page with database, translating and adding recipes. Testing flash messages and bodyparser. Removing idea to add upload users own images, too much work.|
|21st Nov. 	|10 hours|	Working with the browse page, testing to fetch DB data. |
|24th Nov. 	|10 hours|	Testing designs on browse fetching parts from DB and also routing chosen recipes, creating the chosenrecipe page and designing. |
|27th-28th Nov.| 	15-20 hours|	Adding more space on both sides of the design, empty columns.  Working with the design, trying to make it more responsive. |
|4th Dec. 	|8 hours|	Working on design of recipe cards and recipe page, images and checking code.|
|8th Dec.	|9 hours|	Cleaning up code, adding more comments to describe functionality, publishing on Render. |
|11th Dec. 	|10 hours	|Working on re-writing documentation, planning and adding info describing the project|
|Total	110-120 hours|||	

# 5	Database Structure
The RecipeApp project relies on a MongoDB database to store and manage recipe data. The database, named "RecipeApp," is designed to efficiently organize and retrieve recipe information. In this section, we will explore the structure of the MongoDB database and its collections.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/a2e75a50-cb27-4aa1-a241-983a5f3ef182)


The central collection in the RecipeApp database is named "recipes." This collection stores individual recipe entries, each represented as a document with specific fields. Here's an overview of the fields within the "recipes" collection:
- _id: A unique identifier automatically generated by MongoDB for each recipe entry. This field serves as the primary key for the documents.
- name: The name of the dish, which provides a concise title for the recipe.
- description: A detailed description of how the dish is made. This field contains the step-by-step in-structions for preparing the recipe.
- ingredients: An array containing the list of ingredients required for the recipe. Each ingredient is stored as a string within the array.
- category: The category to which the recipe belongs. It helps in classifying recipes into different groups, such as "Baking," "Meat," "Vegan," or "Dessert."
- __v: A version key added by MongoDB's Mongoose library to handle document versioning. This field helps track changes to documents over time.

# 6	Web App Functionality
The app.js file serves as the entry point of the web application and is responsible for configur-ing the Express.js server. It imports various modules and middleware to handle different as-pects of the application. One of the middleware used is express-session, which enables session management for user authentication and flash messages. The dotenv middleware is used to load environment variables from the .env file, enhancing security by keeping sensitive data like database connection strings out of version control.

To establish a connection to the MongoDB database, the mongoose module is used. The mon-goose connection is initiated in the database.js file, which is located in the models directory. This file defines the database schema and connection details. It exports the established con-nection, allowing other parts of the application, including controllers and routes, to interact with the database using Mongoose models.

In the controllers directory, the recipeController.js file defines route handlers for various end-points related to recipes. These handlers use Mongoose models and the mongoose connection established in database.js to perform CRUD (Create, Read, Update, Delete) operations on the database. For example, when a user submits a new recipe through the web interface, the cre-ateRecipe function in recipeController.js handles the request by creating a new recipe docu-ment in the MongoDB database.

App.js also configures EJS view engine for rendering dynamic web pages. It sets the layout file for EJS views to './layouts/main', which specifies the structure and layout of the rendered HTML. This layout file allows for consistent headers, footers, and navigation menus across dif-ferent pages of the application, ensuring a unified user experience.

The view engine is set to 'ejs', indicating that EJS will be used to render dynamic content within the layout structure. EJS enables the inclusion of JavaScript code within HTML templates, allow-ing for dynamic data rendering and presentation.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/b033270d-cf5c-4c5e-9181-cf912aabb434)

Additionally, the app.js file sets up routes for different parts of the application using Ex-press.js's routing system. For instance, the /browse route is mapped to the browseRecipes func-tion in the recipeController.js file, which retrieves recipes from the database and renders them on the browse page.
Overall, the app.js file, along with middleware, connects the different parts of the application, allowing it to handle user requests, interact with the database, and render dynamic web pages based on user input. This architecture separates concerns, making the codebase organized and maintainable.

## 6.1	Interface

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/b17e40b7-5954-415c-ba64-83e0e60d3919)

Starting at the homepage, the main layout is used – as for all other pages on the app – and the homepage contents are injected to it using EJS. I ended up with a design as shown above, trying to keep it as simple as possible. From the home page navigation from the nav-bar and more visible buttons can be used for redirection to the browse or create page. 

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/08ff0a73-1e19-4cbb-8ba4-874655107172)
The browse page, as shown in a phone resolution in Figure 8, fetches the recipes already added from the database and parses them in their own cards vertically aligned down the page. It con-tains an image based on the category, a title and a short description based on the description stored in the database. 

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/2d2ba945-8cea-41d5-b344-005ab2386617)

The Create page contains a form where the user can insert the name, description of how to make, add ingredients and selecting category, before submitting the recipe to be added in the database. Having a clientside javascript connected to the + Ingredient button is also vital, so the user can click the button to add more fields to fill in more ingredients. 
The code selects DOM elements using 'getElementById' and 'querySelector'. It sets up an event listener on a button ('addIngredientsButton').    When the button is clicked, it clones the first 'ingredientDiv', clears its input value, and appends it to the 'ingredientList'

# 7	REST API Endpoints
In this section, I describe the technical details of the RecipeApp's REST API. The API serves as the backbone for communication between the client-side application and the server, facilitating data retrieval and manipulation. Below, it is provide documentation for each REST endpoint, along with essential code snippets highlighting key components of their functionality.

## 7.1	GET / and GET /about
Description & Functionality: The GET / and GET /about endpoint handles GET requests to the root path, rendering the application's homepage for users to explore recipes, and the about page with some project info. These endpoints interact with the index.ejs and about.ejs tem-plates to render the pages. 

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/51c7d2fc-1a8e-4c87-8f0e-d2eb1a149793)

The homepage itself describes the purpose of the page briefly and adding navigation to the browse- and add-recipe sections of the application. The about page serves as a backstory to the web application.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/431eec2a-997b-44b0-a310-a125d1f472e7)


## 7.2	GET /browse
Description & Functionality: The GET /browse endpoint serves GET requests to the /browse path, providing users with access to the browse page. This page offers a column of recipe cards, show with category types such as Baking, Meat, Vegan, and Dessert as well as the dish name and a short description.

When users navigate to the /browse path, the server's response is characterized by the follow-ing actions. It initiates a query to the database, specifically utilizing Recipe.find(), to retrieve a collection of recipes. To help the user experience, the server also integrates two essential help-er functions: getRecipeImage and shortenDescription. These helpers serve a dual purpose:

getRecipeImage: Dynamically determines and attaches images to each recipe card based on its category. This ensures that the browse page showcases mouthwater-ing visuals that correspond to each recipe.

shortenDescription: Condenses lengthy recipe descriptions into succinct previews, optimizing space on the recipe cards while providing a brief glimpse into the culi-nary wonders offered.

Finally, the server uses the res.render method to render the browse.ejs template. This template forms the foundation of the browse page, which is adorned with a visually pleasing display of recipe cards. Alongside the cards, users encounter striking images, thanks to the getReci-peImage helper, and concise descriptions that spark their curiosity, courtesy of the shortenDe-scription function.


## 7.3	GET /chosenrecipe/:id
The GET /chosenrecipe/:id endpoint serves as a gateway for users to access comprehensive details about a particular recipe. Upon receiving a user request, the server extracts the :id parameter from the URL, allowing it to pinpoint the specified recipe. Subsequently, a database query is initiated using Recipe.findById(recipeId) to retrieve the in-depth recipe information.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/c7948e18-0a9a-4f12-90f3-1dc2f178f4ac)

In the event that the desired recipe is not found, the server responds with a 404 Not Found error. However, when the requested recipe is successfully located, the server proceeds to render the chosenrecipe.ejs template, which serves as the canvas for presenting the details from the database for the selected recipe. To enhance the visual appeal of the page, the getRecipeImage helper-function finds and helps displaying a category-relevant image. 

## 7.4	GET /create 
Description & Functionality: The GET /create endpoint handles GET requests for accessing the recipe creation form. It renders a form where users can input and submit new recipes. This endpoint interacts with the create.ejs template, serving as the starting point for adding new recipes. As part of its functionality, this endpoint utilizes flash messages to enhance user inter-action. When users access this page, they can encounter two types of flash messages, depend-ing on the context:

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/623c9ecf-e9a8-42b5-85ee-fae2b4dabbd4)

Flash Error Messages: In situations where errors occur during form submission or database op-erations, flashError messages are utilized to communicate the issue to the user. These messag-es offer clear and informative feedback, helping users identify and address any problems.

Flash Success Messages: Following a successful recipe addition, flashCreate messages are em-ployed to inform users of the successful creation of their recipe. These messages serve as con-firmations, enhancing user satisfaction and providing instant feedback on the action's outcome. The endpoint proceeds to render the "create.ejs" template, designed to accommodate these flash messages. It effectively communicates the purpose of the page, allowing users to input and submit new recipes. The incorporation of flash messages ensures that users are promptly informed of any errors and successful recipe additions, contributing to an intuitive and user-friendly recipe creation process.

## 7.5	POST /create

Description: 
The POST /create endpoint is a fundamental component of the RecipeApp, responsible for processing POST requests submitted by users when they create and submit a new recipe. This endpoint handles the intricate process of taking user-provided data, such as the recipe's name, description, ingredients, and category, and integrates it into the application's database.

![image](https://github.com/Kjersheim/recipe-webapp/assets/90902439/4a01c73c-199a-4343-9994-ff6f16a69e38)

Functionality:
Recipe Object Creation: Upon receiving a POST request to /create, the server invokes the rec-ipeController.createAdd function. Inside this function, a new Recipe object is created using data extracted from the request body. This includes the recipe's name, description, ingredients, and category.

Database Integration: The newly created Recipe object, referred to as newRecipe, is prepared for integration into the database. The controller invokes the await newRecipe.save() method, which effectively saves the recipe to the RecipeApp's MongoDB database. This step ensures that the user's recipe submission becomes a permanent part of the application's recipe collec-tion, directly able to be viewed by other users. In a real life application I would add moderation and approval to this for security reasons.
Flash Messages: As described in the previous section about GET /create.

Redirection: After successfully saving the new recipe to the database, the controller employs the res.redirect('/create') method to redirect the user back to the "create.ejs" page. This serves as a confirmation of the recipe addition and allows users to continue adding more recipes if desired.
Error Handling: In the event of an error during recipe creation or database operations, the con-troller logs the error and generates a flashError message, providing informative feedback to the user. The user is then redirected back to the "create.ejs" page, where they can address the is-sue or seek assistance.

# 8	Helper Functions
The project incorporates two essential helper functions, namely getRecipeImage and shortenDescription, which significantly contribute to enhancing the user experience and inter-face of the application. These helper functions serve distinct yet complementary purposes, aimed at improving the presentation and visual appeal of the recipes showcased within the application.

getRecipeImage Function: The getRecipeImage function help in dynamically retrieving and dis-playing recipe images based on their respective categories. It accepts two parameters: category and context. The category parameter represents the category of a recipe, such as "Baking," "Meat," "Vegan," or "Dessert," while the context parameter defines the context in which the function operates, with a default value of '/'.
Internally, the function employs a switch statement to determine the appropriate image URL based on the provided category. For instance, if a recipe belongs to the "Baking" category, the function constructs the image URL using the baseUrl and appends "img/baking.jpg" to it. This process ensures that the image displayed on the recipe card aligns with the category, providing users with a visually cohesive and engaging experience.

shortenDescription Function: The shortenDescription function focuses on optimizing the presentation of recipe descriptions in the browse page's recipe cards. Given the potential length of recipe descriptions, the function aims to create concise and visually appealing pre-views while maximizing the efficient use of space. To achieve this, the function accepts a de-scription parameter representing the full recipe description.
Inside the function, a maximum length, denoted as maxLength, is defined to establish the threshold at which descriptions should be shortened. If a description exceeds this length, the function truncates it to fit within the specified limit and appends an ellipsis ('...') to indicate that the full description continues beyond the preview. By doing so, users browsing the recipe cards are provided with essential insights into the recipe's content without overwhelming them with excessive text.

Both helper functions are seamlessly integrated into the application's templates and rendering processes. getRecipeImage enhances the visual appeal of recipe cards, while shortenDescription ensures that descriptions remain concise and user-friendly. Together, these helper functions contribute to creating an engaging and efficient user interface, enriching the overall RecipeApp experience.

# 9	Strengths, Weaknesses, Final Reflections 
The RecipeApp project demonstrates several strengths and weaknesses in its current form. A notable strength lies in its use of a modern technology stack, including Node.js, Mongoose, EJS, and MongoDB, which offers robust community support and adaptability. The app's user-centric design ensures an intuitive and friendly user experience, complemented by a responsive layout that adapts well to different devices and screen sizes. Additionally, its clear API documentation simplifies backend understanding and integration.
On the flip side, the app exhibits some weaknesses. Its functionality, while adequate for a pro-ject, lacks the depth and breadth of features typically found in commercial-grade applications, such as user accounts and social sharing options. Scalability could be a concern, as the current setup might not efficiently handle a significant increase in concurrent users or data volume. In addition, I would have added a step between the creation of new recipes to the database where added data is moderated and approved/rejected or edited for the purpose of displaying on the webpage. 
Reflecting on the project as a whole, it has been a rewarding endeavor. However, certain challenges, including time constraints and limitations in design creativity and skill, were encountered. Collaboration within a team or soliciting feedback from users for evaluating color palettes and layout choices would have been beneficial. Personally, I would have preferred a cleaner and lighter user interface. These insights highlight opportunities for improvement in future projects, emphasizing the importance of fostering collaboration and refining design elements to enhance overall project quality. 

Considering the time spend on the project and the attempt to incorporate the various challenges from each of the earlier exercises, where they would fit, I believe the project was quite a good solution and personally I would grant it – considering the prerequisite and lower skill levels required on the course – a given strong 1 point or weak 2. Knowing the strong or weak is not a thing that matters on paper I would add it as a 2 point due to use of modules and overall functionality. 






