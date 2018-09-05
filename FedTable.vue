<template>
	<div id="fed-table-wrapper">
		<div class="level">
			<div class="level-left">
				<button @click="csvDownload" v-if="downloadable == 'true'" class="button">
					CSV
				</button>
			</div>
			<div class="level-right">
				<div class="level-item">
					<input class="input is-rounded" @keyup="search()" v-model="searchText" type="text" placeholder="Search">
				</div>
			</div>
		</div>
		<table class="table is-bordered is-striped is-narrow is-hoverable is-fullwidth">
			<thead>
				<th v-for="header in originalHeadersFromJson" @click="sort(header)">
					{{header.trim()}}
					<template v-if="currentSort == header">
						<i v-if="currentSortDir === 'asc'" class="arrow up"></i>
						<i v-if="currentSortDir === 'desc'" class="arrow down"></i>
					</template>

				</th>
			</thead>
			<tbody>
				<tr v-for="cat in sortedRows">
					<td v-for="cell in cat">{{cell}}</td>
				</tr>
			</tbody>
		</table>
		<p>
			<button class="button" @click="prevPage"><</button>
			<input class="input fed-small-input" type="number" v-model="currentPage">
			<button class="button" @click="nextPage"> ></button>
		</p>
		<span> Page {{currentPage}}  out of {{totalPages}} ({{rowCount}} Records)</span>
	</div>
</template>

<script>
    import axios from "axios"
    import Papa from "papaparse"
    export default {
        name: "FedTable",
        props: {
            'dataUri': {
                default: null
            }, 'setHeaders': {
                default: null
            }, 'downloadable': {
                default: true
            }, 'filename': {
                default: 'download.csv'
            },
            'pageSize': {
                default: 15
			}
        },
        data() {
            return {
                rows: null,
                filteredRows: [], //for when using the search bar and exporting to csv
                currentSort: '',
                currentSortDir: 'asc',
                currentPage: 1,
                pages: 0,
                headers: [],
                searchText: ''
            }
        },
        methods: {
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
                this.filteredRows = [];
                if (this.searchText.trim() !== '') {
                    var that = this;
                    this.rows.forEach(function (e) {
                        var jsonRow = JSON.stringify(e);
                        if (jsonRow.includes(that.searchText)) {
                            that.filteredRows.push(e)  //move filtered rows to a new array
                        }
                    })
                }


            },
            nextPage: function () {
                if (this.currentSet < this.rowCount) this.currentPage++;
            },
            prevPage: function () {
                if (this.currentPage > 1) this.currentPage--;
            },
            csvDownload: function () {
                var data = Papa.unparse(this.filteredRows); //all json data to csv
                var csvData = new Blob([data], {type: 'text/csv;charset=utf-8;'});
                var csvURL = null;
                if (navigator.msSaveBlob) {
                    csvURL = navigator.msSaveBlob(csvData, this.filename);
                } else {
                    csvURL = window.URL.createObjectURL(csvData);
                }
                var tempLink = document.createElement('a');
                tempLink.href = csvURL;
                tempLink.setAttribute('download', this.filename);
                tempLink.click();
            },

        },
        computed: {
            sortedRows: function () {
                var searchArray = [];
                //use filtered if search bar is in use
                if (this.searchText != '' && this.searchText !== null) {
                    searchArray = this.filteredRows
                } else {
                    this.searchText = null;
                    searchArray = this.rows
                }

                //sort based on asc or desc
                return searchArray.sort((a, b) => {
                    let modifier = 1;
                    if (this.currentSortDir === 'desc') modifier = -1;
                    if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
                    if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
                    return 0;
                }).filter((row, index) => { //only show results for current page number
                    let start = (this.currentPage - 1) * this.pageSize;
                    let end = this.currentPage * this.pageSize;
                    if (index >= start && index < end) return true;
                });
            },
            originalHeadersFromJson: function () {
                if (this.rows != null) {
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
                return Math.ceil(this.rows.length / this.pageSize)
            }
        },
        created() {
            this.getData();
        },
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
</style>
