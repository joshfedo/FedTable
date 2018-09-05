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
					<input class="input is-rounded" @keyup="search()" v-model="searchBar" type="text" placeholder="Search">
				</div>
			</div>
		</div>
		<table class="table is-bordered is-striped is-narrow is-hoverable is-fullwidth">
			<thead>
				<th v-for="header in pulledHeaders" @click="sort(header)">
					{{header.trim()}}
					<template v-if="currentSort == header">
						<i v-if="currentSortDir === 'asc'" class="arrow up"></i>
						<i v-if="currentSortDir === 'desc'" class="arrow down"></i>
					</template>

				</th>
			</thead>
			<tbody>
				<tr v-for="cat in sortedCats">
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
    import VueCsvDownloader from 'vue-csv-downloader'
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
            }
        },
        data() {
            return {
                cats: null,
                filteredCats: [],
                currentSort: '',
                currentSortDir: 'asc',
                pageSize: 15,
                currentPage: 1,
                pages: 0,
                headers: [],
                searchBar: ''
            }
        },
        methods: {
            getData: function () {
                axios.get(this.dataUri).then(reponse => {
                    this.cats = reponse.data
                })
            },
            sort: function (s) {
                //if s == current sort, reverse
                if (s === this.currentSort) {
                    this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc';
                }
                this.currentSort = s;
                this.currentPage = 1;
            },
            search: function () {
                this.filteredCats = [];
                if (this.searchBar.trim() !== '') {
                    var that = this;
                    this.cats.forEach(function (e) {
                        var jsonRow = JSON.stringify(e);
                        if (jsonRow.includes(that.searchBar)) {
                            that.filteredCats.push(e)
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
                var data = Papa.unparse(this.filteredCats);
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
            sortedCats: function () {

                var searchArray = [];
                if (this.searchBar != '' && this.searchBar !== null) {
                    searchArray = this.filteredCats
                } else {
                    this.searchBar = null;
                    searchArray = this.cats
                }


                return searchArray.sort((a, b) => {
                    let modifier = 1;
                    if (this.currentSortDir === 'desc') modifier = -1;
                    if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
                    if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
                    return 0;
                }).filter((row, index) => {
                    let start = (this.currentPage - 1) * this.pageSize;
                    let end = this.currentPage * this.pageSize;
                    if (index >= start && index < end) return true;
                });
            },
            pulledHeaders: function () {
                if (this.cats != null) {
                    return Object.keys(this.cats[0])
                }
            },
            rowCount: function () {
                if (this.searchBar !== null) {
                    return this.filteredCats.length;
                }
                return this.cats.length;

            },
            currentSet: function () {
                return (this.currentPage * this.pageSize)
            },
            totalPages: function () {
                // return Math.celi(this.pageSize/this.rowCount)
                return Math.ceil(this.cats.length / this.pageSize)
            }
        },
        created() {
            this.getData();
        },
        components: {
            VueCsvDownloader
        }
    }
</script>

<style scoped>
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
