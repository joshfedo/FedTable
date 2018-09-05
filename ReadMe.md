##FedTable

###Install 
```npm install vue-fed-table```

###usage
```
//index.js
var app = new Vue({
    el:"#app",
    components:{
        'fed-table': FedTable
    }
});
```

```
#index.html
<div id="app">
    <fed-table data-uri="https://jsonplaceholder.typicode.com/photos"></fed-table>
</div>
```

###Options
| Prop         | Type     | Description                                                       | Required? | Default Value | Example                                                |   |
|--------------|----------|-------------------------------------------------------------------|-----------|---------------|--------------------------------------------------------|---|
| dataUri      | String   | URI to json data                                                  | Yes       | null          | data-uri="https://jsonplaceholder.typicode.com/photos" |   |
| downloadable | Boolean  | Addes a CSV download button                                       | No        | true          | downloadable="true"                                    |   |
| filename     | String   | Name of file that will be downloaded if downloable is set to true | No        | download.csv  | filename="yourFile.csv"                                |   |
| pageSize     | Int      | How many records to show per page                                 | No        | 15            | pageSize="10"                                          |   |

