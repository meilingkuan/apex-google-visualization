# apex-google-visualization
Automatically exported from code.google.com/p/apex-google-visualization
https://developer.salesforce.com/page/Google_Visualizations

GVizGoogleVisualizations.jpg

An ever-increasing list of visualizations can be found in the Google Visualization Gallery. We’ve also shared a project on developer.force.com/codeshare that makes it dead simple for Force.com developers to utilize Google visualizations in their Visualforce pages. This article will discuss how to use the code found in that project.
Installation
The first thing that you'll need to do is install the Google Visualizations project from Code Share in to your Developer Edition org. The instructions for doing so are here.
Architecture
The diagram below illustrates the different components of a Visualforce page that utilizes a custom Google Visualization component. It’s important to notice that in order to embed a Google Visualization in a Visualforce page, you only need to provide the blue-colored portions of the diagram. Namely, a Visualforce page that will display the visualization and an Apex controller that will query the data for the visualization. The components themselves and their supporting classes are provided for you.

GVizArchitecture.jpg

System Interactions
One of the interesting aspects of Google Visualizations is that they are rendered client-side in the user’s browser. This means that none of your sensitive data is ever sent outside of the Force.com platform.

GVizSystemInteractions.jpg

The diagram above shows that when a user requests a Visualforce page containing a Google Visualization component, the Visualforce page will call a method from the page’s controller. This method will return a dataset in the form of a JSON string. This page will then be returned to the user’s browser. The page will then instruct the users’s browser to download the appropriate visualization JavaScript from Google’s servers. Once downloaded, the user’s browser will use the visualization JavaScript to process and render the JSON-formatted data.
Sample Page
Here’s a sample Visualforce page with a Timeline visualization on it:

GVizTimeline.jpg

Generated JSON

The data behind the above timeline is a 2-dimensional table:
Date	Cosmo G. Spacely	Account Name	Opportunity Amount
7/1/2008	1	Robot Maid Co.	$500,000
7/7/2008	2	Flying Cars, Inc.	$300,000
…	…	…	…
In JSON -format, that looks like:
01
{cols: [
02
      {id: "col1", label: "Date", type: "date"},
03
      {id: "col2", label: "Cosmo G. Spacely", type: "number"},
04
      {id: "col3", label: "Account Name", type: "string"},
05
      {id: "col4", label: "Opportunity Amount", type: "string"}
06
       ],
07
 rows: [
08
        {c:[{v: new Date(2008, 7, 1, 0, 0, 0), f: "7/1/2008"},
09
            {v: 1.0},
10
            {v: "Robot Maid Co."},
11
            {v: 500000.0}]},
12
        {c:[{v: new Date(2008, 7, 7, 0, 0, 0), f: "7/7/2008"},
13
            {v: 2.0},
14
            {v: "Flying Cars, Inc."},
15
            {v: 300000.0}]},
16
     …
17
       ]
18
}
Generated HTML

If you're curious, the HTML generated by the Visualforce page can be viewed here.
Google Visualization JavaScript

And the (obfuscated) visualization JavaScript sent by Google can be viewed here.
Visualforce Page
Let’s dig a little deeper to understand how the Visualforce page, the Google Visualization component and the Controller generate the above HTML.
01
<apex:page controller="SalesController" showheader="false">
02
<html>
03
   <body>
04
 
05
      Number of Sprockets Sold By Rep<br/>
06
 
07
    <c:TimeLine jsondata="{!SalesActivity}"
08
                width="900px" height="300px"
09
                zoomStartTime="new Date(2008, 07, 01)"
10
                zoomEndTime="new Date(2008, 09, 30)"
11
                displayZoomButtons="false"
12
                colors="'#C2CD23'" />
13
 
14
   </body>
15
</html>
16
</apex:page>
As you can see, there’s not too much going on in the VF page, just a few of the component parameters being set. The most interesting part of the page is the call to the getSalesActivity() method of the controller.
Apex Controller
The getSalesActivity() method in the controller example below does five things:
Instantiates a GoogleViz object
Creates columns with the appropriate supported data type. (See the 'cols property' section of the Google Visualization DataTable documentation for more information on the supported data types.)
Queries the Opportunity table for all of the current User’s Opportunities.
Creates a row of data for each Opportunity.
Calls the toJsonString() method from the GoogleViz class to return the dataset as a JSON-formatted string.

01
public class SalesController {
02
 
03
    public String getSalesActivity(){
04
        GoogleViz gv = new GoogleViz();
05
                 
06
        gv.cols = new list<GoogleViz.col> {
07
            new GoogleViz.Col('col1','Date','date'),
08
            new GoogleViz.Col('col2', UserInfo.getFirstName() + ' ' +
09
                                      UserInfo.getLastName(), 'number'),
10
            new GoogleViz.Col('col3','Account Name','string'),
11
            new GoogleViz.Col('col4','Opportunity Amount','string')
12
        };
13
                 
14
        Integer numOpportunities = 1;
15
         
16
        for( Opportunity o : [SELECT Id, Name, Amount, CloseDate, Account.Name, Owner.Name
17
                              FROM Opportunity
18
                              WHERE IsWon = true AND OwnerId = :UserInfo.getUserId()
19
                              ORDER BY CloseDate ASC]){
20
 
21
            GoogleViz.row r = new GoogleViz.row();
22
            r.cells.add ( new GoogleViz.cell(o.CloseDate) ); // Date column
23
            r.cells.add ( new GoogleViz.cell(numOpportunities) ); // Quantity column
24
            r.cells.add ( new GoogleViz.cell(o.Account.Name) ); // Annotation Title column
25
            r.cells.add ( new GoogleViz.cell(o.Amount) ); // Annotation Text column
26
 
27
            gv.addRow( r );
28
 
29
            numOpportunities++;
30
        }
31
 
32
        return gv.toJsonString();
33
    } 
34
}
You’ll notice that there isn’t any JSON-specific code in the controller though. This is because the JSONObject.cls Apex class included in the project takes care of the formatting for you. Your controller just has to extract the relevant data and place it in the GoogleViz object.
And that's it! With just a very small amount of Visualforce and Apex code, it is now possible to embed valuable and insightful visualizations inside your Visualforce pages.
Additional Visualizations
If you’re feeling adventurous, you can take a look at the code for the Visualforce components. You'll see that the complexity level is rather low and that it would be quite easy to add additional components to the project – which you can contribute back to the community via the Code Share project.
Force.com as a Google Visualization Data Source
In addition to being able to embed embed Google Visualizations in your Force.com applications, you can also expose some of your Force.com data in a way that is compliant with the Google Visualization API. This lets your carefully crafted public data drive Google Gadgets and Google Visualizations on your own website - or on any other website that supports embedding the gadgets and visualizations. Learn more about exposing data on Force.com to support the Google Visualization API.
Updating Your Google Visualizations Project
This project is periodically updated with code improvements and new components. To update your Google Visualization project in the future, follow the instructions described here.
Related Content
Code Share Project
Google Visualization API
Google Visualization Gallery
Google Visualization Playground
Force.com as a Google Visualization Data Source

https://developer.salesforce.com/page/Google_Visualizations_Install_Instructions

Steps to Checkout
Install Subclipse

Create a Force.com project configured for your Developer Edition org.

In the Force.com IDE, choose the SVN Repository Exploring perspective in the upper right-hand corner.
If this is not available, go to the Window menu, select "Open Perspective", choose "Other", then from the menu that appears, select "SVN Repository Exploring".

Right-click in the SVN Repository Explorer and select New -> Repository Location.

Specify the URL of the repository:
http://apex-google-visualization.googlecode.com/svn/trunk/

Click Finish.

Expand the SVN repository until you see the GoogleVisualizations folder.

Right-click on the src folder and choose Export.

For the export directory, click the Browse button and navigate to the project in the file system. (You should select the project directory in this step, and not the src sub-directory.)

Click 'Ok'.

Use the pull-down menu in the upper right-hand corner of the IDE to switch to the Force.com perspective.
If this is not available, go to the "Window" menu, select "Open Perspective", choose "Other", from the menu that appears, select "Force.com".

Right click the src folder in your IDE project and select Refresh. This will update your workspace with the files retrieved from SVN.

Right click the src folder again, and select Force.com > Save to Server. This will update the server with the files retrieved from SVN.

Verify that your project builds without error.

Troubleshooting 
If you have trouble accessing the http://apex-google-visualization.googlecode.com/svn/trunk repository, it may be that your Eclipse environment is behind a proxy server. In this case, go to (Win7) C:\Users\myusername\AppData\Roaming\Subversion directory and edit the servers file. Uncomment and edit (as appropriate for your environment):
http-proxy-host
http-proxy-port
http-proxy-username
http-proxy-password
See SVN doc for details
