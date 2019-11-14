---
layout: page
title: HTML
permalink: /google-html/
---

## HTML 5

- HTML5 is the most recent version of the HTML(Hypertext Markup Language). 
    - It is a language for structuring and displaying content for the World Wide Web, a core technology of the Internet.
- WHATWG (Web Hypertext Application Technology Working Group) another gathering of W3C individuals felt that W3C is not giving careful consideration to this present reality improvement needs of dialect, it has begun dealing with the new determination of HTML-HTML5. Consequently, HTML5 is another adaptation of HTML 4.01 and XHTML 1.0 concentrating on the necessities of Web application designers and in addition tending to issues found in the present details.
- Specifically, HTML5 includes numerous new syntactical features. New elements , like <section>, <article>, <header>, and <nav>, are the essential parts of semantic substance of documents. These additionally incorporate the <video>, <audio>, and <canvas> tags, as well as the integration of SVG content. These components are intended to make it simple to incorporate and handle interactive media and graphical substance on the web without resorting to restrictive modules and APIs. While a few components and traits have been expelled. A few components, for example, <a>, <cite> and <menu> have been changed, redefined or standardized. The APIs and DOM are no longer reconsideration but are fundamental parts of the HTML5 specification.
- The < !DOCTYPE> is an instruction to the web browser about what version of HTML the page is written in. 
    - AND The < !DOCTYPE> tag does not have an end tag and It is not case sensitive.
    - The < !DOCTYPE> declaration must be the very first thing in HTML5 document, before the tag. 
    - As In HTML 4.01, all < ! DOCTYPE > declarations require a reference to a Document Type Definition (DTD), 
    - because HTML 4.01 was based on Standard Generalized Markup Language (SGML). 
    - Where as HTML5 is not based on SGML, and therefore does not require a reference to a Document Type Definition (DTD).
- Some of the most interesting new features in HTML5:
    - The <canvas> element for 2D drawing
    - The <video> and <audio> elements for media playback
    - Support for local storage
    - New content-specific elements, like <article>, <footer>, <header>, <nav>, <
- Advantages of HTML5 :
    - Mutuality
    - Cleaner mark-up / Improved Code
    - Improved Semantics
    - Elegant forms and web apps
    - Offline Application cache
- Canvas is a element of HTML5 which uses JavaScript to draw graphics on a web page. A canvas is a rectangular area. Each and every pixel of it can be controlled by us. There are several methods for drawing paths, boxes, circles, characters, and adding images by using canvas.
- To add canvas tag to our HTML document we need id, width and height. Below is the example how to write a basic canvas tag to your HTML document.
~~~html
<canvas id="myFirstCanvas" width="100" height="100"> </canvas>
~~~
- HTML5 nGeolocation is used to locate a user’s position
- The HTML5 Geolocation API is used to get the geographical position of a user.
- Since this can compromise user privacy, the position is not available unless the user approves it.
- Use the getCurrentPosition() method to get the user’s position.
- The example below is a simple Geolocation example returning the latitude and longitude of the user’s position:
~~~html
<script>
var x = document.getElementById("demo");
function getLocation()
{
    if (navigator.geolocation)
    {
        navigator.geolocation.getCurrentPosition(showPosition);
    }
    else{
        x.innerHTML="Geolocation is not supported by this browser.";
    }
}

function showPosition(position)
{
    x.innerHTML="Latitude: " + position.coords.latitude +"
    Longitude: " + position.coords.longitude;
}
</script>
~~~
- Rules for HTML5 :
    - New features should be based on HTML, CSS, DOM, and JavaScript
    - Reduce the need for external plugins (like Flash)
    - Better error handling
    - More markup to replace scripting
    - HTML5 should be device independent
    - The development process should be visible to the public.
- The sessionStorage object stores the data for one session. The data is deleted when the user closes the browser window. like below we can create and access a sessionStorage here we created “blogName” as session
~~~html 
<script type="text/javascript">
    sessionStorage.blogName="OnlineInterviewQuestions";
    document.write(sessionStorage.name);
</script>
~~~
- The new HTML5 specification allows browsers to prefetch some or all of website assets such as HTML files, images, CSS, JavaScript, and so on, while the client is connected. It is not necessary for the user to have accessed this content previously, for fetching this content. In other words, application cache can prefetch pages that have not been visited at all and are thereby unavailable in the regular browser cache. Prefetching files can speed up the site’s performance, though you are of course using bandwidth to download those files initially.
- List of new API’s provided by the Html 5 standard.
    - Canvas : Canvas consists of a drawable region defined in HTML code with height and width attributes. JavaScript code may access the area through a full set of drawing functions similar to other common 2D APIs, thus allowing for dynamically generated graphics. Some anticipated uses of the canvas include building graphs, animations, games, and image composition.
    - Timed media playback
    - Offline storage database
    - Document editing
    - Drag-and-drop
    - Cross-document messaging
    - Browser history management
    - MIME type and protocol handler registration
- Before HTML5 LocalStores was done with cookies. Cookies are not very good for large amounts of data, because they are passed on by every request to the server, so it was very slow and in-effective.
- In HTML5, the data is NOT passed on by every server request, but used ONLY when asked for. It is possible to store large amounts of data without affecting the website’s performance.and The data is stored in different areas for different websites, and a website can only access data stored by itself.
- And for creating localstores just need to call localStorage object like below we are storing name and address
~~~html 
<script type="text/javascript">
    localStorage.name="ABC";
    localStorage.address="New Delhi India.";
    document.write(localStorage.address);
</script>
~~~
- Input type Attribute in HTML5
    - tel The input is of type telephone number
    - search The input field is a search field
    - url a URL
    - email One or more email addresses
    - datetime A date and/or time
    - date A date
    - month A month
    - week A week
    - time The input value is of type time
    - datetime-local A local date/time
    - number A number.
    - range A number in a given range.
    - color A hexadecimal color, like #82345c
    - placeholder Specifies a short hint that describes the expected value of an input field.
    
## HTML 
- HTML is a widely used language that gives our webpage a structure. 
    - HTML stands for HYPERTEXT MARKUP LANGUAGE. 
    - HTML was created by Berners lee in 1991. 
    - A markup language is a set of markups tags. 
    - Every webpage is created in HTML. 
    - HTML documents are described by HTML tags.
    - This language is basically used for creating web applications and also for web pages as well. 
    - It is a standard markup language with cascading style sheets and JavaScript which form a triad for WWW i.e. worldwide Web.
    - HTML came into existence in 1980 when a great computer professor Sir Tim Berners Lee (a contractor and author of HTML) proposed an idea in CERN, to basically sharing and using documents. 
- Berners prototyped ENQUIRE (a system for CERN), which would help researchers to use and share documents.
    - HTML Tags was basically the first publicly description of HTML which was first mentioned by Tim Berners Lee in late 1991. 
    - He is usually the professor of computer programming. 
    - He is the author of HTML which is treated to be as the foremost computer languages of all the times. 
    - His idea of writing this language brought a tycoon in the computer technology world and also nowadays it is also much popular and most trending.
- Sir Tim Berners-Lee, a great physicist who is a great personality is the author of HTML. 
    - He is also the director of the World Wide Web Consortium i.e. W3C. 
    - He is also the founder of WWWF i.e.World wide Web Foundation. 
        - Open Data Institute is founded by him and also he is director of this institute.
        - He is a director of the Web Science Research Initiative i.e. WSRI and a member of the advisory board of the MIT Center for Collective Intelligence. 
        - Sir Tim Berners is a great personality, as he is the professor of the computer department and also the founder of many institutes and Research initiatives. 
        - An idea struck into his mind, how can a computer be made helpful for using and sharing documents. 
        - So, he designed the most trending and tremendous language, through which it is very easy to make web pages and all. 
        - Also, it is most trending and still is in use rather like other outdated languages.
- The sequence of characters or other symbols which we usually use to insert at certain places in a word or text file in order to indicate how the file should look when it is displayed or printed or Basically to describe documents logical structure. The markup indicators are often known as tags. Elements and tags are two different words which we need to understand as there is a lot of confusion among public related to this fact. It must be noted that HTML documents contain tags, but do not contain the elements. It is a wide misconception that usually occurs in mind that both exist in HTML documents.elements are usually only generated after parsing step from these tags
- A Physical tag has physical text which is used to tell the browser how to display the text enclosed in the physical tag.
- Example for the physical tags are: <big>, <b>, <i>
- Logical tags are used to tell the meaning of the enclosed text in it. The example of the logical tag is <Important>….</Important> tag. When we enclose text in Important tag then it tell the browser that enclosed text is more important than other text.
- If you go and have a look at HTML markup for any webpage today, you will let to know that HTML elements are contained within other HTML elements. These elements which are inside of other are known as nested elements and they had become the essential part of building any web page nowadays. The most expertise way to know more about nesting is just to think about HTML tags in the form of boxes that hold your content which can be in form of text, images,etc.HTML tags are basically the boxes around the content.
- XHTML means Extensible Hypertext Markup Language, which is basically a part of Family of XML markup language. It usually extends the most popularly used HTML i.e. Hypertext Markup Language, the pages in which the web pages are formulated.
- Cell Spacing is referred to space/gap between the two cells of the same table.
- Cell Padding is referred to the gap/space between the content of the cell and cell wall or Cell border.
- Example:
- <table border cellspacing=3>
- <table border cellpadding=3>
- <table border cellspacing=3 cellpadding=3>
- People always confuse in understanding terms elements and tags in HTML. It must be noted that HTML documents contain tags, but do not contain the elements. It is a wide misconception that usually occurs in mind that both exist in HTML documents.elements are usually only generated after parsing step from these tags.
- An HTML element is basically an individual component usually of an HTML  web page or document. Once if this has been also parsed into the document object model.HTML nodes are the main part that composes HTML for example such as text nodes.HTML attributes can be specified from each node. Content, including other nodes and text, together with build up a node. Semantics or meanings are mostly represented by many HTML nodes. Let us take an example of a title node represents the title of the document.
- In HTML, we have two types of lists unordered lists and ordered lists. Unordered list starts with <ul> tag and ends with </ul> tag. Ordered tag starts with <ol> and ends with </ol>. Each list item is written as <li></li>
- Basically, when we usually start with a hyperlink that is Mailto rather than using HTTP or others, the browser should be able to compose an email in the visitor’s default email program on their computer. There are two ways to insert a mailto link into a HubSpot page or email, just by inserting a hyperlink into the rich editor or inputting into the source code. Option one is basically nothing just to insert a mailto link as a hyperlink, also highlight the text or click on the image as you wish.to apply on the mailto link to for your contacts or visitors to click on and finally click Add the link to finish. Option two is to Insert a mailto link into the source code and Click Save to finish then Publish or Update your page.
- Example:
- <a href="mailto:someone@example.com?Subject=Hello%20again" target="_top">Send Mail</a>
- Question 14) What does DOCTYPE mean?
- DOCTYPE or Document Type Declaration is a type of instruction which usually works in association with particular SGML or XML documents basically. Let us take an example to understand it more thoroughly, for example, A Web page with a document type definition i.e. DTD is the best to understand. In a well serialized and a proper form of the document, It manifests and also the contribution of it is a lot as a short string of markup that usually conforms to a particular syntax.
- Question 15) List the media types and formats supported by HTML ?
- HTML supports a wide range of media formats for sound, music, videos, movies, and animations. Below is the list extensions supported by each media format.
- Media Type	Formats Supported
- Images	png, jpg, jpeg, gif, apng, svg, bmp, BMP ico, png ico
- Audio	MIDI, RealAudio, WMA, AAC, WAV, Ogg, MP3, MP4
- Video	MPEG, AVI, WMV, QuickTime, RealVideo, Flash, Ogg, WebM, MPEG-4 or MP4
- HTML supports 6 types of heading tags they are H1, H2, H3, H4, H5, and H6. H1 is the highest and most important heading tag.
- Semantic HTML or Semantic Markup is HTML that introduces meaning to the web page rather than just presentation.
- <form>, <table>, and <article> are examples of Semantic Elements.
- Below are the list of few new Semantic Elements introducted HTML5
    - <article>
    - <aside>
    - <details>
    - <figcaption>
    - <figure>
    - <footer>
    - <header>
    - <main>
    - <mark>
    - <nav>
    - <section>
    - <summary>
    - <time>
- Tim Berner Lee(Tim Berners-Lee) is known as the father of "World Wide Web".
- HTML tag is just opening or closing entity.
    - Example: <p> and </p> are called HTML tags
    - HTML element encompasses opening tag, closing tag, content (optional for content-less tags)
    - Example: <p>This is the content</p> : This complete thing is called an HTML element
- Grouping is used to group several HTML controls like input, textarea, selects as well as labels ( <label>) within a web form. In HTML <fieldset> element is used for Grouping.
- The <fieldset> is a tag in HTML that is used to group related elements in a form. It draws a box around the related elements.
- The difference between span and div is that a span element is in-line and usually used for a small chunk of HTML inside a line (such as inside a paragraph) whereas a div (division) element is block-line (which is basically equivalent to having a line-break before and after it) and used to group larger chunks of code.
- <div id="scissors">
-     <p>This is <span class="paper">crazy</span></p>
- </div>
- Div is a division tag in HTML. It helps us to divide our page into different sections. Div is a block element.
- Syntax
- <div> Your content</div>
- A block level element in HTML always starts with a new line on document and expand to full width of the page or container. <address>, <p>,<dir>,<div>,<figure>, <header> are few examples of block level elements in HTML.
- Empty tags in html are tags with no content.Empty tag element are singular tag and no closing tag is required. Examples of empty tags are <br/>, <link>,<meta>,<image>,<hr/> etc
- Inline text or inline elements does not cause a line break on a HTML Page or container they takes only space bounded by its opening and closing tag. <span>, <a>, <b>,<input>, <kbd>, <label> are few examples of inline elements in HTML.
- Id attribute in HTML is used to uniquely identify an element in HTML DOM. You can use id attribute as a CSS/jquery selector to style or perform some dom related action on element.
- An unordered list (<ul>) is used to create a bulleted list in HTML. Below is a simple example to create bulleted list in HTML.
- <p>Fruits List</p>
- <ul>
- <li>Apple</li>
- <li>Banana</li>
- <li>Mango</li>
- </ul>
- Head tag (<head>) in Html is used to specify important information (metadata) about certain webpage.Common elements placed between <head> tag of html are <base>, <link>, <meta>, <noscript>, <script>, <style> <title>
- An HTML document is a file used for displaying Hypertext, media, and information on web. It is written in hypertext markup language and contains HTML tags with information.HTML document is either saved with .html or .htm extension.
- The basic structure of HTML document looks like
- <!DOCTYPE html>
- <html>
-     <head>
-         <title>Your Page Title</title>
-     </head>
-     <body>
-         <h1>First Heading</h1>
-         <p>Paragraph Text.</p>
-     </body>
- </html>
- The basic purpose of using alternative texts is to define what the image is about. During an image mapping, it can be confusing and difficult to understand what hotspots correspond to a particular link. These alternative texts come in action here and put a description at each link which makes it easy for users to understand the hotspot links easily.
- Image mapping lets a user to link one image to different web pages in and out of the website. It is the process of defining special shapes inside an image and link it to different destinations.
- Yes, Collapsing white spaces is one good way of writing HTML codes in an organized way. White spaces act similar to a single-spaced character and occupy the same size as rest of the characters. By collapsing multiple white spaces, we can indent lines of text and make the code more readable.
- Copyright symbol can be either copied and pasted from other sources. Or, you can also add it by writing a small piece of code: &copy; or & #169; in the required page
- HTML DOM is an object model for HTML and Application Programming Interface(API) for Javascript to work with HTML elements, properties and their methods.    