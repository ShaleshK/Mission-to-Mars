### Wrote code to append a table to a web page and adds a row with a UFO sighting from a dataset (data.js) in the form of an array of javascript Objects.

![UFO Zoom-In]("UFO.JPG")
![Table]("Filter.JPG")

### HTML code for zoom-in animation
```html
    <div class="hero text-center">
      <h1>UFO Sightings</h1>
      <p>The Truth is Out There</p>
    </div>
```
### HTML code for the table
```html
          <div id="table-area" class="">
            <table id="ufo-table" class="table table-striped">
              <thead>
                <tr>
                  <th class="table-head">Date</th>
                  <th class="table-head">City</th>
                  <th class="table-head">State</th>
                  <th class="table-head">Country</th>
                  <th class="table-head">Shape</th>
                  <th class="table-head">Duration</th>
                  <th class="table-head">Comments</th>
                </tr>
              </thead>
              <tbody></tbody>
            </table>
```
### JavaScript buildtable function
```javascript
var tbody = d3.select("tbody");
function buildTable(data) {
    tbody.html("");
    data.forEach(function(ufoSightings) {
        console.log(ufoSightings);
        var row = tbody.append("tr");
    
        Object.entries(ufoSightings).forEach(function([key, value]) {
               console.log(key, value);
               var cell = row.append("td"); 
               cell.text(value);
        });
    });
}
```

### JavaScript code that will listen for events and search through the `date/time` column to find rows that match user input
```Javascript
var button = d3.select("#filter-btn");
button.on("click", function() {
      var inputElement = d3.select("#datetime");
      var inputValue = inputElement.property("value");      
      console.log(inputValue);
      console.log(tableData);
      var filteredData = tableData;
    //   filteredData = filteredData.filter(row => row.datetime === inputValue);

      Object.entries(filters).forEach(([key,value]) => {
        filteredData = filteredData.filter(row => row[key] === value);
      }); 

      console.log(filteredData);
      buildTable(filteredData);
});
buildTable(tableData);
```
### javascript filter function
```javascript
var filters = {};
function updateFilters() {
    var inputElement = d3.select(this).select("input");
    var inputValue = inputElement.property("value");
    var filterID = inputElement.attr("id");
    if (inputValue) {
        filters[filterID] = inputValue;
    } else {
        delete filters[filterID]
    }
}
d3.selectAll(".filter").on("change",updateFilters);
```

### filter in html
```html
<div class="panel-body">
                <form>
                  <div class="form-group">
                    <ul class="list-group" id="filters">
                      <li class="filter list-group-item">
                        <label for="date">Enter a Date</label>
                        <input class="form-control" id="datetime" type="text" placeholder="1/11/2011">
                      </li>
                      <li class="filter list-group-item">
                        <label for="city">Enter a City</label>
                        <input class="form-control" id="city" type="text" placeholder="el cajon">
                      </li>
                      <li class="filter list-group-item">
                        <label for="state">Enter a State</label>
                        <input class="form-control" id="state" type="text" placeholder="ct">
                      </li>
                      <li class="filter list-group-item">
                        <label for="country">Enter a Country</label>
                        <input class="form-control" id="country" type="text" placeholder="usa">
                      </li>
                      <li class="filter list-group-item">
                        <label for="shape">Enter a Shape</label>
                        <input class="form-control" id="shape" type="text" placeholder="circle">
                      </li>
                    </ul>
                  </div>
                  <button id="filter-btn" type="button" class="btn btn-default">Filter Table</button>
                </form>
              </div>
```