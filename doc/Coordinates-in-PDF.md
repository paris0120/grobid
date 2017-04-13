 <h1>Coordinates of structures in the original PDF</h1>

## Getting the coordinates of identified structures

### Limitations

As of April 2017, coordinate areas can be obtained for the following document substructures: 

* ```persName``` for a complete author name
* ```figure``` for figure AND table
* ```ref``` for bibliographical, figure, table and formula reference markers - for example (`Toto and al. 1999`), see `Fig. 1`, as shown by `formula (245)`, etc.
* ```biblStruct``` for a bibliographical reference

However, there is normally no particular limitation to the type of structures which can have their coordinates in the results, the implementation is on-going, see [issue #69](https://github.com/kermitt2/grobid/issues/69), and it is expected that more or less any structures could be associated with their coordinates in the orginal PDF. 

Coordinates are currently available in full text processing (returning a TEI document) and the PDF annotation services (returning JSON). 

### GROBID service

To get the coordinates of certain structures extracted from a PDF document, use in the query the parameter ```teiCoordinates``` with the list of structures than should have coordinates in the results. Possible structures are indicated above. 

Example with cURL:

* add coordinates to the figures (and tables) only:

> curl -v --form input=@./12248_2011_Article_9260.pdf --form teiCoordinates=figure --form teiCoordinates=biblStruct localhost:8080/processFulltextDocument

* add coordinates for all the supported elements (sorry for the ugly cURL syntax on this):

> curl -v --form input=@./12248_2011_Article_9260.pdf --form teiCoordinates=persName --form teiCoordinates=figure --form teiCoordinates=ref --form teiCoordinates=biblStruct localhost:8080/processFulltextDocument


### Batch processing

We recommand to use the above service mode for best performance and range of options. 

Generating coordinates can also been obtained with the batch mode by adding the parameter ```-teiCoordinates``` with the command ```processFullText```.

Example: 

```bash
> java -Xmx1024m -jar grobid-core-0.4.1.one-jar.jar -gH /path/to/Grobid/grobid/grobid-home -gP /path/to/Grobid/grobid-home/config/grobid.properties -dIn /path/to/input/directory -dOut /path/to/output/directory -teiCoordinates -exe processFullText 
```

See the [batch mode details](http://grobid.readthedocs.io/en/latest/Grobid-batch/#processfulltext). With the batch mode, it is currenlty not possible to cherry pick up certain elements, coordinates will appear for all.


## Coordinate system in the PDF

### Generalities

The PDF coordinates system has three main characteristics: 

* contrary to usage, the origin of a document is at the upper left corner. The x-axis extends to the right and the y-axis extends downward,

* all locations and sizes are stored in an abstract value called a PDF unit,

* PDF documents do not have a resolution: to convert a PDF unit to a physical value such as pixels, an external value must be provided for the resolution.

In addition, contrary to usage in computer science, the index associated to the first page is 1 (not 0).

### Coordinates in JSON results

The processing of a PDF document by the GROBID result in JSON containing two specific structures for positioning entity annotations in the PDF :

* the list of page size, introduced by the JSON attribute __pages__. The dimension of each page is given successively by two attributes page_height and page_height.

Example: 
```json
	"pages": [ {"page_height":792.0, "page_width":612.0}, 
 	{"page_height":792.0, "page_width":612.0}, 
	{"page_height":792.0, "page_width":612.0}, 	
```

* for each entity, a json attribute __pos__ introduces a __list of bounding boxes__ to identify the area of the annotation corresponding to the entity. Several bounding boxes might be necessary because a textual mention does not need to be a rectangle, but the union of rectangles (a union of bounding boxes), for instance when a mention to be annotated is on several lines.

Example: 
```json
"pos": [{  "p": 1,
			"x": 20,
			"y": 20,
			"h": 10,
			"w": 30 }, 
		{  "p": 1,
			"x": 30,
			"y": 20,
			"h": 10,
			"w": 30 } ]
```

A __bounding box__ is defined by the following attributes: 

- __p__: the number of the page (beware, in the PDF world the first page has index 1!), 

- __x__: the x-axis coordinate of the upper-left point of the bounding box,

- __y__: the y-axis coordinate of the upper-left point of the bounding box (beware, in the PDF world the y-axis extends downward!),

- __h__: the height of the bounding box,

- __w__: the width of the bounding box.

As a PDF document expresses value in abstract PDF unit and do not have resolution, the coordinates have to be converted into the scale of the PDF layout used by the client (usually in pixels). This is why the dimension of the pages are necessary for the correct scaling, taking into account that, in a PDF document, pages can be of different size. 

The GROBID console offers a reference implementation with PDF.js for dynamically positioning entity annotations on a processed PDF. 

### Coordinates in TEI/XML results

Coordinates for a given structure appear as an extra attribute ```@coord```. This is part of the [customization to the TEI](TEI-encoding-of-results.md) used by GROBID.

Similarly as for JSON, the coordinates of a structure is provided as a list of bounding boxes, each one seprated by a semicolon ```;```, each bounding box being defined by 5 attributes separated by a comma ```,```:

Example 1: 
```xml
<author>
	<persName coords="1,53.80,194.57,58.71,9.29"><forename type="first">Ron</forename><forename type="middle">J</forename><surname>Keizer</surname></persName>
</author>
```

"1,53.80,194.57,58.71,9.29" indicates one bounding box with attributes page=1, x=53.80, y=194.57, h=58.71, w=9.29.

Example 2:
```xml
<biblStruct coords="10,317.03,183.61,223.16,7.55;10,317.03,192.57,223.21,7.55;10,317.03,201.53,223.15,7.55;10,317.03,210.49,52.22,7.55"  xml:id="b19">
```

The above ```@coords``` XML attributes introduces 4 bounding boxes to define the area of the bibliographical reference (typically because the reference is on several line).

As side note, in traditionnal TEI an area should be expressed using SVG. However it would have make the TEI document quickly unreadable and extremely heavy and we are using this more compact notation. 