## FedTable
FedTable is a simle sourtable, searchable table writen in Vue js. Simply pass in your JSON array via a url and the component will do the rest! This project is styled using Bulma. 

[logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 2"


### Install 
```npm install vue-fed-table```

### usage
```javascript
//index.js
import Vue from 'vue'
import FedTable from 'vue-fed-table'

var app = new Vue({
    el:"#app",
    components:{
        'fed-table': FedTable
    }
});
```

```html
#index.html
<div id="app">
    <fed-table data-uri="https://jsonplaceholder.typicode.com/photos"></fed-table>
</div>
```

### Options
| Prop         | Type     | Description                                                       | Required? | Default Value | Example                                                |   |
|--------------|----------|-------------------------------------------------------------------|-----------|---------------|--------------------------------------------------------|---|
| dataUri      | String   | URI to json data                                                  | Yes       | null          | data-uri="https://jsonplaceholder.typicode.com/photos" |   |
| downloadable | Boolean  | Addes a CSV download button                                       | No        | true          | downloadable="true"                                    |   |
| filename     | String   | Name of file that will be downloaded if downloable is set to true | No        | download.csv  | filename="yourFile.csv"                                |   |
| page-size     | Int      | How many records to show per page                                 | No        | 15            | pageSize="10"                                          |   |

