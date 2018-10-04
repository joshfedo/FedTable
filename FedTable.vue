<template>
	<div id="fed-table-wrapper">
		<div class="level">
			<div class="level-left">
				<div class="level-item">
					<button @click="csvDownload" v-if="downloadable == 'true'" class="button">
						CSV
					</button>

				</div>
				<div class="level-item">
					<button @click="print()" v-if="printable == true" class="button">
						Print
					</button>
				</div>
			</div>
			<div class="level-right">
				<div class="level-item">
					<input class="input is-rounded" @keyup="search()" v-model="searchText" type="text" placeholder="Search">
				</div>
			</div>
		</div>
		<table v-bind:class="{table:!printing, 'is-bordered':!printing,'is-striped':!printing, 'is-narrow':!printing,'is-hoverable':!printing, 'is-fullwidth':!printing}">
			<thead>
			<th v-for="(header, index) in tableHeaders" @click="sort(header)">
				{{displayHeaders[index]}}
				<template v-if="currentSort == header">
					<i v-if="currentSortDir === 'asc'" class="arrow up"></i>
					<i v-if="currentSortDir === 'desc'" class="arrow down"></i>
				</template>
			</th>

			</thead>
			<tbody>
			<tr v-for="cat in sortedRows">

				<template v-for="(cell,index) in cat">
					<td v-if="index.toLowerCase().includes('date')">{{cell | formatDate}}</td>
					<td v-else-if="index.toLowerCase().includes('note')">
						<button class="button" @click="getNote(cat,$event)"  v-bind:class="{'is-link':cell != null,'is-secondary': cell == null}">Note</button>

					</td>
					<td v-else>{{cell }}</td>


				</template>
			</tr>
			</tbody>
		</table>
		<p>
			<button class="button" @click="prevPage"><</button>
			<input class="input fed-small-input" min="1" v-bind:max="totalPages" type="number" v-model="currentPage">
			<button class="button" @click="nextPage"> ></button>
		</p>
		<span> Page {{currentPage}}  out of {{totalPages}} ({{rowCount}} Records)</span>
	</div>
</template>

<script>
    import axios from "axios"
    import Papa from "papaparse"
    import EventBus from "./EventBus"
    import FedNote from "./FedNote";
    import Vue from 'vue';
    import VueSweetalert2 from 'vue-sweetalert2';
    import moment from 'moment'
    Vue.use(VueSweetalert2);


    export default {
        name: "FedTable",
        components: {FedNote},
        props: {
            'dataUri': {
                default: null
            }, 'setHeaders': {
                default: null
            }, 'downloadable': {
                default: true
            }, 'fileName': {
                default: 'download.csv'
            }, 'pageSize': {
                default: 15
            }, 'noteId': {
                default: null
            }, 'printable':{
                default: true
            }, 'notes' :{
                default: true
            }, 'reportId' :{
                default: 'null'
            }, 'currentSort' :{
                default: null
            }, 'dateColumns' :{
                default: null
            }
        },
        data() {
            return {
                rows: null,
                filteredRows: [], //for when using the search bar and exporting to csv

                currentSortDir: 'asc',
                currentPage: 1,
                pages: 0,
                headers: [],
                searchText: '',
                printing: false,
                pageSizeBuffer : 0,
            }
        },
        methods: {
            hasTable:function(){
                axios.get("/notesAPI/hasTable/" + this.reportId)

            },
            getData: function () {
                axios.get(this.dataUri).then(response => {
                    this.rows = response.data
                })
            },
            sort: function (s) {
                //change sort direction
                if (s === this.currentSort) {
                    this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc';
                }
                this.currentSort = s;
                this.currentPage = 1; //rest page to 1 after reordering
            },
            search: function () {

                if(this.currentPage > this.totalPages){
                    this.currentPage = 1;
                }

                this.filteredRows = [];
                if (this.searchText.trim() !== '') {
                    var that = this;
                    this.rows.forEach(function (e) {
                        var jsonRow = JSON.stringify(e);
                        if (jsonRow.toLowerCase().includes(that.searchText.toLowerCase())) {
                            that.filteredRows.push(e)  //move filtered rows to a new array
                        }
                    })
                    if(this.filteredRows.lenght == 0){
                        this.filteredRows = [{results:'No Results Found'}]
                    }
                }


            },
            nextPage: function () {
                if (this.currentSet < this.rowCount) this.currentPage++;
            },
            prevPage: function () {
                if (this.currentPage > 1) this.currentPage--;
            },
            csvDownload: function () {
                if(this.filteredRows.length != 0){
                    var data = Papa.unparse(this.filteredRows); //all json data to csv
                }else{
                    var data = Papa.unparse(this.rows);
                }

                var csvData = new Blob([data], {type: 'text/csv;charset=utf-8;'});
                var csvURL = null;
                if (navigator.msSaveBlob) {
                    csvURL = navigator.msSaveBlob(csvData, this.fileName);
                } else {
                    csvURL = window.URL.createObjectURL(csvData);
                }
                console.log(csvData)
                var tempLink = document.createElement('a');
                tempLink.href = csvURL;
                tempLink.setAttribute('download', this.fileName);
                tempLink.click();
            },
            getNote: function (row,$event) {
                //get the note id for this row that is used in the table
                var idKeys = this.noteId.split(',');
                var compoundId = '';
                idKeys.forEach(function(e){
                    compoundId += row[e]
                })

                //if we already have a note pop the show and add, else pop add note
                axios.get("/notesAPI/get/" + this.reportId,{params:{'id':compoundId}}).then(result=>{

                    let allNotes = result.data
                    if (allNotes.length > 0){
                        console.log(allNotes)
                        var swalBody = '<table class="table is-fullwidth">'
                        for( var i = 0; i < allNotes.length; i++){
                            swalBody += "<tr><td style='font-size: small'>" + allNotes[i]['note'] + "</td><td style='font-size: small'>" + allNotes[i]['time'] +"</td></tr>"
                        }


                        this.$swal.setDefaults({
                            title: 'Note',
                            html: swalBody,
                            confirmButtonText: 'Add Note',
                            showCancelButton: true,
                            cancelButtonText: 'Close',
                        })
                        var steps = [
                            'Notes',
                            {title:'Add Note',
                                input:'textarea'
                            }
                        ]

                        this.$swal.queue(steps).then((result) => {
                            this.$swal.resetDefaults()

                            if (result.value) {

                                axios.get('/notesAPI/set/'+ this.reportId,{params:{
                                        'id': compoundId,
                                        'note': result.value[1]
                                    }})
                            }})

                    }else{
                        var text = this.$swal({
                            title: 'Add Note',
                            input: 'textarea',
                            inputPlaceholder: 'Type your message here',
                            showCancelButton: true
                        }).then((result)=>{
                            console.log(result['value'])
                            if (result.value) {

                                axios.get('/notesAPI/set/'+ this.reportId,{params:{
                                        'id': compoundId,
                                        'note': result.value
                                    }})

                                console.log($event)
                                $event.target.classList.toggle('is-link')

                            }
                        }  )
                    }

                })

            },
            print: function () {
                this.currentPage = 1;
                this.printing = true;
                this.pageSizeBuffer = this.pageSize
                this.pageSize = 9999;

                Vue.nextTick(function () {
                    window.print()
                })






                // window.print()
                //
            },
            printEvent(){
                this.pageSize = this.pageSizeBuffer
                this.printing = false
            },

        },
        computed: {
            sortedRows: function () {
                var searchArray = [];
                //use filtered if search bar is in use
                if (this.searchText != '' && this.searchText !== null) {
                    searchArray = this.filteredRows;

                } else {
                    this.searchText = null;
                    searchArray = this.rows
                }
                //sort based on asc or desc
                return searchArray.sort((a, b) => {
                    if (this.currentSort == this.dateColumns){
                        console.log('sorting by a date ')
                        a = moment(a)
                        b = moment(b)
                        if (a > b){
                            return 1
                        }else if (a < b){
                            return -1
                        }else{
                            return 0
                        }
                    }else{
                        let modifier = 1;
                        if (this.currentSortDir === 'desc') modifier = -1;
                        if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
                        if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
                        return 0;
                    }

                }).filter((row, index) => { //only show results for current page number
                    let start = (this.currentPage - 1) * this.pageSize;
                    let end = this.currentPage * this.pageSize;
                    if (index >= start && index < end) return true;
                });
            },
            tableHeaders: function () {
                if (this.rows != null) {
                    return Object.keys(this.rows[0])
                }
            },
            displayHeaders: function(){
                if(this.setHeaders != null){
                    return this.setHeaders.split(',')
                }else{
                    return Object.keys(this.rows[0])
                }
            },
            rowCount: function () {
                if (this.searchText !== null) {
                    return this.filteredRows.length;
                }
                return this.rows.length;
            },
            currentSet: function () {
                return (this.currentPage * this.pageSize)
            },
            totalPages: function () {
                // return Math.celi(this.pageSize/this.rowCount)
                return Math.ceil(this.rowCount / this.pageSize)
            }
        },
        created() {
            this.getData();
            this.hasTable();
            window.addEventListener('afterprint', this.printEvent);



        },
        destroyed(){
            window.removeEventListener('afterprint', this.handleScroll)
        },
        filters:{
            formatDate(value){
                let unix  = moment(value*1000)
                if (unix.format('MM/DD/YY') == '12/31/69'){
                    return ''
                }
                return unix.format('MM/DD/YY')

            }
        }
    }
</script>

<style scoped>
	@import 'bulma.css';


	#fed-table-wrapper {
		padding: 15px;
	}

	.fed-small-input {
		width: 80px;
	}

	i {
		border: solid black;
		border-width: 0 3px 3px 0;
		display: inline-block;
		padding: 3px;
	}

	.up {
		transform: rotate(-135deg);
		-webkit-transform: rotate(-135deg);
	}

	.down {
		transform: rotate(45deg);
		-webkit-transform: rotate(45deg);
	}


	table, td, th {
		border: 1px solid #ddd;
		text-align: left;
	}

	table {
		border-collapse: collapse;

	}

	th, td {
		padding: 3px;
	}


</style>
