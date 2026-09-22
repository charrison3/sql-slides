AutoAnalysis v4


One Query Per Chart
A question of whether or not I really need the functionality of one query for all charts in a more optimized database. This might not really be a problem in the new world and it’s possible that it never really was as much of a problem as I made it out to be.

 The thing about this is that I do really love the ability run the SQL as you got it without having to use from third-party UI. Even though the parameterization aspect of it does force non-runnable SQL I want to maintain this style of user interface as long as I can since it is really light weight and decently elegant. Once I abandon this I am violating my core principle and am just like any other BI tool 

Duck DB + UI
Additionally, I might incorporate duck DB as an essential element of my process. This probably only makes sense if I have some other UX for writing SQL. I would need a compiler that takes fully valid SQL but persists any CTE in a duck db table. Currently the biggest problem with the auto analysis is that it is way slower to build charts. If I could add this section I could drag and drop the charts and see data returned instantly. The PowerPoint would be secondary. The drag and drop part could be distinct from the chart vis part, it would just build chart CTEs

JSON over tabular
The other core idea here would be to get the SQL output in JSON instead of tabular Union data. The benefit here is that it wouldn’t require any specific format and you could mix data for tables multi series charts etc. We could also structure this in a way where there is an extremely limited level of transformation logic. There would not need to be any jntermediate data structures, we could transform directly into ChartData objects. They could even be a world where we would for all of the auto analysis code off of Python– PPTX. Or we could make some modification distinction between auto analysis objects, and Python – PPTX objects in the input JSON.

I looked into this a good bit more and didn’t really find any smoking gun with JSON. The one object is instead of select star in the union I could do object construct star and this would allow me to name my keys. Anything I want, however, the output format would be pretty odd because it would be one column with one row per row of data rather than one per of chart.

Omni Over Auto Analysis
Another thing that I should try and find out is using Omni as much as possible, the more I can stay with Omni the more I can contribute naturally to its data model. If I can create a workflow making charts, I got a commentary on the side maybe 


Claude prompt 
Create a document in this chat to call out these potential refactors. Use the anthropic style slide copying system to simplify the slide duplication system that currently exists today add short code snippets to explain what we found 

Also add a section on refactoring to take advantage of Python-pptx’s extension point for parts of my code that fit well into that 

Create a more detailed section on all of the learnings from the chat that will help inform the creation of a stable method of at first replacing data on combo charts 


AI with skills and without code 

How to Build This Version
1. Go all in on the coding style and process skills and documentation 
    * Add test suites?
2. Wire frame the most basic directory structure 
3. Really go crazy with the documentation for each feature 
4. Build a test dataset (or steal a CSV)
5. Go all in on the ability to copy from deck to deck
    1. Do deep analysis of what corrupts a deck and what does not 
    2. Start with the simplest copy and progress 
6. Build the Core 
7. Build the VisData
8. Connect Omni using ChartDataframes 
9. Build the TemplateManager

Rename it SQLSlides
The package that connects the most powerful data analysis languages to the most elegant data storytelling tool.

All in on template first logic
If we could duplicate slides across decks we could massively simplify the code as we right now have a from scratch method that is completely unique from the copy template side of things. After this the scope of the package would ONLY be on replacing data, not on building charts from scratch 

In this world, you could make any type of crazy chart you wanted combo or otherwise and save it as a default layout with your custom name for it. These names would define what is refencible in the SQL

Ideally we could store every single one of the configs in the deck. There would be a helper package to create a starting point for your master template deck. You insert your layout and it spits out a deck with one slide per each of the basic supported charts. This would allow us to get away with crazy config slides like a color palette slide with squares or a text box slide with text presets or even an excel format slide with names and : numbers or the actual format idc. Probably one with chart colors and fonts. The nice thing about this deck is that it would explain how to use the package.


VisData Class
This optional helper class would replace the current split charts data structure. Maybe we could set it up where you don’t need this and can just dump ChartDataframes 

It could accept input data in a variety of formats and convert it all into the chart data that is required by Python-PPTX.
* ChartDF
* TableDF
* KPI
* Omni query
* Plotly Chart (accept raw chart, use functions to format it?)

Not sure but maybe TableDataframes could be SIX pivoted columns of table ID, row number, row label (optional), value and value format.

KPIs could just be three columns KPI ID, value and value format. I think value format could be an f string format or an excel format if I find a good translator since that allows $ and millions more easily. KPIs search the deck looking for {{ kpi_id }}. KPIs could be used for any text including chart titles or axis labels. For long streaks of open text it is expected that you do that in post processing 

There would be an append df method or append data (agnostic of data type) or an append method called on the instance variable allowing you to append to any data type

This could be entirely decoupled from the rest of the code which would be a cool benefit. I could do all of the data preprocessing that I do currently in there. Additionally, I could finally introduce the “data calculations” feature that I have been dreaming of for a while. The Vis Data class could have all sorts of functionality for referencing individual charts or sets of charts either by the title, title contains or the characteristics of the data. When running a calculation you could choose duplicate or in place. For duplicate you could do things like copy and change vis type or copy and year over year with the same data 

Def period over period(date_granularity)
   ….

yoy_charts = get_charts(filter_func = if “over time” in chart.title, calculation = period_over_period(year), is_duplicate=True)


Maybe filter func could accept a dataframe ID string or a function?? To make it easier 

Some data calcs out of the box could be 
* Top n values 
* Bottom n values 
* Average line (additional line series to make combo chart )
* Running total line
* Running percent line
* Year over year 

Data calcs could also be associated with template slides on a one to many basis. For things with line

More advanced projections would be time series projection (creates another series of the same color with dots)

The vis data class could enrich the ChartDataframes with all sorts of optional columns meaning that we could make some standard columns like segment and pivot order optional (maybe even chart type which could be intuited or defaulted). More importantly we could make all sorts of other columns like combo_series or second_y_axis. This also fundamentally frees us to choose whether to do a given transformation in the SQL or in Python/ 

 This could give us options for how to handle the date casting stuff. The elegant way would be to add a multi axis major and multi axis minor. If we wanted all that fancy auto logic we could make it a data calculation allowing me to decouple that. This would not prevent manual overrides I could still make a multi x axis manually either in the SQL or adhoc in visdata with a function 

We could add a chart id column if the DS didn’t have it you would inherit from the title. This would allow you to divorce the title from the unique ID as early as the data stage. 

Another fun one would be loop group ID. This would indicate that all of those charts belong to one loop. That way if more ever got added it would know to make more from the template. Potentially it could delete them but this behavior would just be inherited from a template calculation

Templates Class
In the same way as the VisData class it would be great to at least mostly pull any template logic into its own class. I don’t think we could completely decouple and frontload this, since there is some runtime data that does edit the templates. That said it could completely own anything to do with templates 

This would hold the template path and the master path. It would handle  the whole weirdness where if no analysis template (path) is provided it creates an empty deck from the master. Builds slides from the master (aka build from CSV) one at a time. This can also be triggered via parameter to happen whenever a new Chart is added to the CSV. There is also maybe an argument for overwrite template 

There are currently two template calculations. One would be combine decks. We had to do all sorts of fancy logic with the slide IDs to make this work which we will have to port over. 

The other one js slide looping. We pass it a slide and then duplicate that x times while changing the bread crumb VisData IDs

To enable advanced features we may need to pass the VisData instance. But I don’t think that there is anything within the core that requires us to do it there 


Core Functionality
This shrinks the core down to a super small responsibility of just replacing data in charts . The core will receive pre transformed data and a ready to go template. All it has to do is swap the data out. I’m sure there will be some other things like swapping in KPIs or fixing up colors 



VisualCalculations
This concept exists today and I like the idea of it whether this really had to do that much or is just an analysis pattern 

This class should help you select any set of slides charts or objects of any sort and apply some sort of function to them on a loop. They can relate to the data or who cares what.

This should run on either a Presentation or a  .pptx file 

