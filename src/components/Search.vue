<template>
<div class="dropdown">
    <div @click="closeDropdown()" v-if="allRecordSites == true || allRecordPages == true ||  magic_flag == true || advancedSearch == true || result_search == true" style="position:absolute;top:0.8rem;right:-10px;cursor: pointer;z-index:1">
        <i class="fa fa-times"></i>
    </div>
    <input v-model.trim="inputValue" @focus="focus()" class="dropdown-input" type="text" placeholder="Search" />
    <!-- all type -->
    <div v-if="magic_flag" class="dropdown-content">
        <div v-for="type in typeSearch" :key="type.id">
            <div @click="showRecordSearch(type.name)" class="type_show">
                <i class="fa fa-id-card"></i>
                <span style="margin-left:1rem">{{type.name}}</span>
            </div>
        </div>
        <div @click="openPopupAdvancedSearch()" class="dropdown-advanced-search">
            <p>Advanced Search</p>
        </div>
    </div>
    <!-- type Sites -->
    <div v-if="allRecordSites" class="dropdown-content">
        <VuePerfectScrollbar v-if="recordsSites.length > 7" class="scroll-area" v-once :settings="settings">
            <div v-for="item in recordsSites" :key="item.id">
                <div class="type_show">
                    <i class="fa fa-id-card"></i>
                    <span style="margin-left:1rem">{{item.name}}</span>
                    <span style="float:right">{{item.time}}</span>
                </div>
            </div>
        </VuePerfectScrollbar>
        <div v-else v-for="item in recordsSites" :key="item.id">
            <div class="type_show">
                <i class="fa fa-id-card"></i>
                <span style="margin-left:1rem">{{item.name}}</span>
                <span style="float:right">{{item.time}}</span>
            </div>
        </div>
    </div>
    <!-- type Pages -->
    <div v-if="allRecordPages == true  && magic_flag == false" class="dropdown-content">
        <VuePerfectScrollbar v-if="recordsPages.length > 7" class="scroll-area" v-once :settings="settings">
            <div v-for="item in recordsPages" :key="item.id">
                <div class="type_show">
                    <i class="fa fa-id-card"></i>
                    <span style="margin-left:1rem">{{item.name}}</span>
                    <span style="float:right">{{item.time}}</span>
                </div>
            </div>
        </VuePerfectScrollbar>
        <div v-else v-for="item in recordsPages" :key="item.id">
            <div class="type_show">
                <i class="fa fa-id-card"></i>
                <span style="margin-left:1rem">{{item.name}}</span>
                <span style="float:right">{{item.time}}</span>
            </div>
        </div>
    </div>
    <!-- advanced search -->
    <div v-if="advancedSearch" class="dropdown-content-advanced">
        <div style="padding :1rem 1rem 0 1rem">
            <span style="color:gray;font-size:15px">Type</span>
            <select @change="typeChange($event)" style="margin-left:5.2rem; width:70%;border:none;border-bottom : 1px solid #e2e8f0" v-model="selected">
                <option value="All Types">All Types</option>
                <option v-for="item in typeSearch" :key="item.id">{{item.name}}</option>
            </select>
        </div>
        <div style="padding :1rem 1rem 0 1rem">
            <span style="color:gray;font-size:15px">Owner</span>
            <select @change="ownerChange($event)" style="margin-left:4.2rem; width:70%;border:none;border-bottom : 1px solid #e2e8f0" v-model="owner">
                <option value="Anyone">Anyone</option>
                <option value="Owner by me">Owner by me</option>
                <option value="Not owner by me">Not owner by me</option>
                <option value="Specific person">Specific person...</option>
            </select>
            <div v-if="specific_person">
                <div v-if="person_default != 'Select person'" @click="closeChoosePerson()" style="text-align:center;height:22px;width:22px;position:absolute;top:8rem;right:0.7rem;cursor: pointer;z-index:1;background-color:#e2e8f0;border-radius:50%">
                    <i class="fa fa-times"></i>
                </div>
                <select @change="personChange($event)" style="margin-left:7rem;margin-top:1rem; width:70%;border:none;border-bottom : 1px solid #e2e8f0" v-model="person_default">
                    <option value="admin">admin</option>
                    <option value="designer">designer</option>
                    <option value="editor">editor</option>
                    <option value="sysadmin">sysadmin</option>
                </select>
            </div>
            <div style="padding: 1rem 0 0 0">
                <input v-on:change="filterStarted()" style="margin-left:7.2rem" type="checkbox" id="start" value="start" v-model="started">
                <label style="padding-left:1rem" for="start">Started</label>
                <input v-on:change="filterInTrash()" style="margin-left:3rem" type="checkbox" id="in_trash" value="in trash" v-model="in_trash">
                <label style="padding-left:1rem" for="in_trash">In Trash</label>
            </div>
        </div>
        <div style="padding :1rem 1rem 0 1rem">
            <span style="color:gray ;font-size:15px">Date Modified</span>
            <select style="margin-left:0.9rem; width:70%;border:none;border-bottom : 1px solid #e2e8f0" v-model="date_modified">
                <option value="Anytime">Anytime</option>
                <option value="today">Today</option>
                <option value="yesterday">Yesterday</option>
                <option value="last7days">Last 7 days</option>
                <option value="last30days">Last 30 days</option>
                <option value="last90days">Last 90 days</option>
            </select>
        </div>
        <div style="padding :1rem 1rem 0 1rem">
            <span style="color:gray;text-overflow:eclips; font-size:15px">Item Name/Title</span>
            <input style="margin-left:5px;position:absolute;width:69%;border:none;border-bottom : 1px solid #e2e8f0">
        </div>
        <div style="padding :1rem 1rem 0 1rem">
            <span style="color:gray;text-overflow:eclips; font-size:15px">Has the words</span>
            <input style="margin-left:17px;position:absolute;width:69%;border:none;border-bottom : 1px solid #e2e8f0" v-model="key_words">
        </div>
        <div style="padding: 2rem 1rem 0 1rem; text-align:end">
            <button class="btn-reset">RESET</button>
            <button @click="actionSearch()" class="btn-search">SEARCH</button>
        </div>
    </div>

    <!-- result search -->
    <div v-if="result_search" class="dropdown-content">
        <div v-if="response_search.length == 0" class="type_show">
            <p>No result find</p>
        </div>
        <div v-else>
            <VuePerfectScrollbar v-if="response_search.length > 7" class="scroll-area" v-once :settings="settings">
                <div v-for="item in response_search" :key="item.id+' '+item.type">
                    <div class="type_show">
                        <i class="fa fa-id-card"></i>
                        <span style="margin-left:1rem">{{item.name}}</span>
                        <span style="float:right">{{item.time}}</span>
                    </div>
                </div>
            </VuePerfectScrollbar>
            <div v-else v-for="item in response_search" :key="item.id+' '+item.type">
                <div class="type_show">
                    <i class="fa fa-id-card"></i>
                    <span style="margin-left:1rem">{{item.name}}</span>
                    <span style="font-size:10px;color:#e2e8f0;padding-left:10px;font-weight:bold">{{item.type}}</span>
                    <span style="float:right">{{item.time}}</span>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
// import axios from 'axios'
import "@fortawesome/fontawesome-free/css/all.css";
import "@fortawesome/fontawesome-free/js/all.js";
import VuePerfectScrollbar from 'vue-perfect-scrollbar'

export default {
    name: 'Search',
    components: {
        VuePerfectScrollbar
    },
    watch: {
        key_words: function () {
            if (this.inputValue != '') {
                this.key_words = this.inputValue;
            } else {
                this.inputValue = '';
                if (this.selected != 'All Types' && this.owner != 'Anyone' && this.started && this.in_trash) {
                    this.inputValue = this.inputValue + "type: " + this.selected + " owner:" + this.owner + " is:started is:in trash " + this.key_words;
                } else if (this.selected != 'All Types' && this.owner != 'Anyone' && this.started) {
                    this.inputValue = this.inputValue + "type: " + this.selected + " owner:" + this.owner + " is:started " + this.key_words;
                } else if (this.selected != 'All Types' && this.owner != 'Anyone') {
                    this.inputValue = this.inputValue + "type: " + this.selected + " owner:" + this.owner + " " + this.key_words;
                } else if (this.selected != 'All Types') {
                    this.inputValue = this.inputValue + "type: " + this.selected + " " + this.key_words;
                } else {
                    this.inputValue = this.inputValue + this.key_words;
                }
            }
        },
        inputValue: function () {
            console.log("DEBUG::: input value: ", this.inputValue);
            if (this.inputValue.length > 0) {
                if (this.inputValue.includes('type:pages'.toLowerCase())) {
                    console.log("DEBUG::: type:pages");
                    this.selected = 'Pages';
                } else if (this.inputValue.includes('type:sites'.toLowerCase())) {
                    this.selected = 'Sites';
                    console.log("DEBUG::: type:sites");
                }else if (this.inputValue.includes('type:all types'.toLowerCase())) {
                    this.selected = 'All Types';
                    console.log("DEBUG::: type:All types");
                }
                 else if (this.inputValue.includes('owner:owner by me'.toLowerCase())) {
                    this.owner = 'Owner by me';
                    this.specific_person = false;
                    this.person_default = 'Select person';
                    console.log("DEBUG::: owner by me");
                } else if (this.inputValue.includes('owner:not owner by me'.toLowerCase())) {
                    this.owner = 'Not owner by me';
                    this.specific_person = false;
                    this.person_default = 'Select person';
                    console.log("DEBUG::: not owner by me");
                } else if (this.inputValue.includes('owner:admin'.toLowerCase())) {
                    this.owner = 'Specific person';
                    this.person_default = 'admin';
                    this.specific_person = true;
                    console.log("DEBUG::: owner admin");
                } else if (this.inputValue.includes('owner:designer'.toLowerCase())) {
                    this.owner = 'Specific person';
                    this.person_default = 'designer';
                    this.specific_person = true;
                    console.log("DEBUG::: owner designer");
                } else if (this.inputValue.includes('owner:editor'.toLowerCase())) {
                    this.owner = 'Specific person';
                    this.person_default = 'editor';
                    this.specific_person = true;
                    console.log("DEBUG::: owner editor");
                } else if (this.inputValue.includes('owner:sysadmin'.toLowerCase())) {
                    this.owner = 'Specific person';
                    this.person_default = 'sysadmin';
                    this.specific_person = true;
                    console.log("DEBUG::: owner sysadmin");
                }else{
                    this.key_words = this.inputValue;
                }
            }
        }
    },
    data() {
        return {
            inputValue: '',
            key_words: '',
            person_default: 'Select person',
            selected: 'All Types',
            owner: 'Anyone',
            started: false,
            in_trash: false,
            date_modified: 'Anytime',
            settings: {
                maxScrollbarLength: 60
            },
            response_search: [],
            typeSearch: [{
                    'id': 1,
                    'name': 'Sites'
                },
                {
                    'id': 2,
                    'name': 'Pages'
                }
            ],
            recordsSites: [{
                'id': 1,
                'name': 'Login',
                'type': 'Sites',
                'time': 'Jan 11, 2019'
            }, {
                'id': 2,
                'name': 'Demo Site',
                'type': 'Sites',
                'time': 'Jan 11, 2019'
            }],
            recordsPages: [{
                    'id': 1,
                    'name': 'HomePage',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 2,
                    'name': 'Login',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 3,
                    'name': 'DemoPage',
                    'type': 'Pages',
                    'time': 'Jan 13, 2019'
                },
                {
                    'id': 4,
                    'name': 'TestPage',
                    'type': 'Pages',
                    'time': 'Jan 12, 2019'
                },
                {
                    'id': 5,
                    'name': 'RegisterPage',
                    'type': 'Pages',
                    'time': 'Jan 13, 2019'
                },
                {
                    'id': 6,
                    'name': 'NotificationPage',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 7,
                    'name': 'SettingsPage',
                    'type': 'Pages',
                    'time': 'Jan 14, 2019'
                },
                {
                    'id': 8,
                    'name': 'MessagePage',
                    'type': 'Pages',
                    'time': 'Jan 15, 2019'
                },
                {
                    'id': 9,
                    'name': 'DetailUser',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 10,
                    'name': 'FindUser',
                    'type': 'Pages',
                    'time': 'Jan 07, 2019'
                },
                {
                    'id': 11,
                    'name': 'NotificationList',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 12,
                    'name': 'NotificationDetails',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 13,
                    'name': 'EventPage',
                    'type': 'Pages',
                    'time': 'Jan 13, 2019'
                },
                {
                    'id': 14,
                    'name': '222222',
                    'type': 'Pages',
                    'time': 'Jan 12, 2019'
                },
                {
                    'id': 15,
                    'name': 'adssd',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                },
                {
                    'id': 16,
                    'name': 'Login',
                    'type': 'Pages',
                    'time': 'Jan 11, 2019'
                }
            ],
            magic_flag: false,
            allRecordSites: false,
            allRecordPages: false,
            advancedSearch: false,
            specific_person: false,
            result_search: false,
        }
    },
    //   mounted () {
    //     this.getList()
    //   },
    methods: {
        focus() {
            if (this.allRecordSites) {
                this.magic_flag = false;
            } else if (this.allRecordPages) {
                this.magic_flag = false;
            } else if (this.advancedSearch) {
                this.magic_flag = false;
            } else if (this.result_search) {
                this.magic_flag = false;
            } else {
                this.magic_flag = true;
                console.log("DEBUG::: flag: ", this.magic_flag);
            }
        },
        noFocus() {
            if (this.allRecordSites) {
                this.magic_flag = false;
            } else if (this.allRecordPages) {
                this.magic_flag = false;
            } else {
                this.magic_flag = false;
                console.log("DEBUG::: flag: ", this.magic_flag);
            }
        },
        showRecordSearch(type) {
            if (type == 'Sites') {
                this.allRecordSites = true;
                this.inputValue = 'type:site';
            } else if (type == 'Pages') {
                this.allRecordPages = true;
                this.inputValue = 'type:page';
            }
            this.magic_flag = false;
            console.log("DEBUG::: show types: ", this.magic_flag);
        },
        openPopupAdvancedSearch() {
            this.advancedSearch = true;
            this.magic_flag = false;
        },
        closeDropdown() {
            this.allRecordSites = false;
            this.allRecordPages = false;
            this.advancedSearch = false;
            this.result_search = false;
            if (this.magic_flag == false) {
                this.magic_flag = true;
            } else {
                this.magic_flag = false
            }
            this.inputValue = '';
            this.response_search = [];
        },
        typeChange(e) {
            if (e.target.value != 'All Types') {
                this.inputValue = " " + this.inputValue + 'type:' + this.selected;
            }
        },
        ownerChange(e) {
            if (e.target.value != 'Anyone') {
                if (e.target.value == 'Specific person') {
                    if (this.selected != 'All Types' && this.started && this.in_trash) {
                        this.inputValue = '';
                        this.inputValue = " " + this.inputValue + 'type:' + this.selected + " is:started " + "is: in trash";
                    } else {
                        this.inputValue = "";
                        if (this.started) {
                            this.inputValue = "";
                            this.inputValue = " " + this.inputValue + 'type:' + this.selected + " is:started ";
                        } else if (this.in_trash) {
                            this.inputValue = '';
                            this.inputValue = " " + this.inputValue + 'type:' + this.selected + " is:in trash ";
                        }
                    }
                    this.specific_person = true;
                } else {
                    this.inputValue = this.inputValue + ' owner:' + this.owner;
                    this.specific_person = false;
                }
            }
        },
        personChange(e) {
            if (e.target.value != 'Select person') {
                this.inputValue = this.inputValue + " owner:" + this.person_default;
            }
        },
        closeChoosePerson() {
            this.person_default = 'Select person';
            if (this.selected != 'All Types' && this.started && this.in_trash) {
                this.inputValue = '';
                this.inputValue = " " + this.inputValue + 'type:' + this.selected;
            } else {
                this.inputValue = "";
                if (this.started) {
                    this.inputValue = "";
                    this.inputValue = " " + this.inputValue + 'type:' + this.selected + " is:started ";
                } else if (this.in_trash) {
                    this.inputValue = '';
                    this.inputValue = " " + this.inputValue + 'type:' + this.selected + " is:in trash ";
                }
            }
        },
        filterStarted() {
            if (this.started) {
                this.inputValue = this.inputValue + " is:" + 'started '
            } else {
                if (this.selected != 'All Types' && this.owner != 'Anyone' && this.in_trash) {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner + " is:in trash ";
                } else if (this.selected == 'All Types') {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + " owner:" + this.owner;
                } else if (this.owner == 'Anyone') {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + "type:" + this.selected;
                } else {
                    this.inputValue = '';
                    if (this.in_trash) {
                        this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner + " is:in trash ";
                    }
                    this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner;
                }
            }
        },
        filterInTrash() {
            if (this.in_trash) {
                this.inputValue = this.inputValue + " is:" + 'in trash '
            } else {
                if (this.selected != 'All Types' && this.owner != 'Anyone' && this.started) {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner + " is: started";
                } else if (this.selected == 'All Types') {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + " owner:" + this.owner;
                } else if (this.owner == 'Anyone') {
                    this.inputValue = '';
                    this.inputValue = this.inputValue + "type:" + this.selected;
                } else {
                    this.inputValue = '';
                    if (this.started) {
                        this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner + " is: started";
                    }
                    this.inputValue = this.inputValue + "type:" + this.selected + " owner:" + this.owner
                }
            }
        },
        actionSearch() {
            this.response_search = [];
            if (this.selected == 'Sites') {
                var sites = this.recordsSites.filter(d => d.name.toLowerCase() === this.key_words.toLowerCase());
                this.response_search = sites;
            } else if (this.selected == 'Pages') {
                var pages = this.recordsPages.filter(d => d.name.toLowerCase() === this.key_words.toLowerCase());
                this.response_search = pages;
            } else {
                var rs_sites = this.recordsSites.filter(d => d.name.toLowerCase() === this.key_words.toLowerCase());
                var rs_pages = this.recordsPages.filter(d => d.name.toLowerCase() === this.key_words.toLowerCase());
                this.response_search = [].concat.apply(rs_sites, rs_pages);
            }
            this.advancedSearch = false;
            this.result_search = true;
        }

    },
}
</script>

<style>
.dropdown {
    position: relative;
    width: 100%;
    max-width: 400px;
    margin: 0 auto;
}

.dropdown-input,
.dropdown-selected {
    width: 100%;
    padding: 10px 16px;
    border: 1px solid transparent;
    background: #edf2f7;
    line-height: 1.5em;
    outline: none;
    border-top-right-radius: 5px;
    border-top-left-radius: 5px;
}

.dropdown-input:focus,
.dropdown-selected:hover {
    background: #fff;
    border-color: #e2e8f0;
}

.dropdown-input::placeholder {
    opacity: 0.7;
}

.dropdown-selected {
    font-weight: bold;
    cursor: pointer;
}

.dropdown-content {
    background-color: white;
    width: 432px;
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
    border: 0.5px solid #e2e8f0;
    border-top: none;
}

.dropdown-content-advanced {
    background-color: white;
    width: 432px;
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
    border: 0.5px solid #e2e8f0;
    border-top: none;
    text-align: left;
    padding-bottom: 1rem;
}

.type_show {
    text-align: left;
    padding: 0.7rem 1rem;
    cursor: pointer;
}

.type_show:hover {
    background-color: #e2e8f0;
}

.dropdown-advanced-search {
    border-top: 1px solid #e2e8f0;
    text-align: left;
    cursor: pointer;
}

.dropdown-advanced-search>p {
    font-size: 13px;
    margin-left: 1rem;
}

.scroll-area {
    position: relative;
    margin: auto;
    height: 300px;
}

.btn-reset {
    background-color: white;
    border: 1px solid #e2e8f0;
    padding: 0.5rem;
    border-radius: 5%;
    color: #3bafee;
    font-weight: bold;
    margin-right: 20px;
}

.btn-search {
    background-color: #3bafee;
    border: 1px solid #3bafee;
    padding: 0.5rem;
    border-radius: 5%;
    color: white;
    font-weight: bold;
}
</style>
