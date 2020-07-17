<!-- > 
    Modified based on vue-csv-import
    GitHub url: https://github.com/jgile/vue-csv-import#readme
<-->
<template>
    <div class="csv-uploader">
        <div class="form-check">
            <input class="form-check-input" type="checkbox" id="headerBox" :value="hasHeaders" @change="toggleHasHeaders" checked>
            <label class="form-check-label" for="headerBox">
                File Has Headers
            </label>
        </div>
        <div class="form-group">
            <input ref="csv" type="file" @change.prevent="validFileMimeType" class="inputClass" name="csv">
            <div class="invalid-feedback d-block" v-if="showErrorMessage">
                File type is invalid
            </div>
        </div>
        <div class="form-group">
            <div class="my-4">
                <v-btn color="primary" :disabled="disabledSubmitButton" @click.prevent="load">
                    Submit
                </v-btn>    
            </div>
        </div>
    </div>
</template>

<script>
    import { get } from 'lodash';
    import Papa from 'papaparse';
    import mimeTypes from "mime-types";

    export default {
        props: {
            fileMimeTypes: {
                type: Array,
                default: () => {
                    return ["text/csv", "text/x-csv", "application/vnd.ms-excel", "text/plain"];
                }
            },
            presentedFields: {
                type: Array,
                default: () => {
                    return ["DO", "TY", "TI", "Y1"];
                }
            },
        },

        data: () => ({
            hasHeaders: true,
            headers: null,
            headerIndex: null,
            detailedHeader: null,
            csv: null,
            detailedInfo: null,
            isValidFileMimeType: false,
            fileSelected: false,
        }),

        methods: {
            toggleHasHeaders() {
                this.hasHeaders = !this.hasHeaders;
            },
            validFileMimeType() {
                let file = this.$refs.csv.files[0];
                const mimeType = file.type === "" ? mimeTypes.lookup(file.name) : file.type;

                if (file) {
                    this.fileSelected = true;
                    this.isValidFileMimeType = this.validateMimeType(mimeType);
                } else {
                    this.fileSelected = false;
                    this.isValidFileMimeType = false;
                }
            },
            validateMimeType(type) {
                return this.fileMimeTypes.indexOf(type) > -1;
            },
            load() {
                const _this = this;
                this.readFile((output) => {
                    _this.csv = get(Papa.parse(output, { skipEmptyLines: true }), "data");
                    if (_this.hasHeaders) {
                        _this.headers = _this.csv[0];
                        _this.csv.shift();
                    }
                    this.headerToObject();
                    this.csvToObject();
                    _this.$emit('inputCSV', _this.csv);
                    _this.$emit('inputHeader', _this.headers);
                    _this.$emit('detailedInput', _this.detailedInfo);
                    _this.$emit('detailedHeader', _this.detailedHeader);
                });
            },
            readFile(callback) {
                let file = this.$refs.csv.files[0];

                if (file) {
                    let reader = new FileReader();
                    reader.readAsText(file, "UTF-8");

                    reader.onload = function (event) {
                        callback(event.target.result);
                    }

                    reader.onerror = function () {
                    };
                }
            },
            headerToObject() {
                let headerArray = this.headers;
                this.headers = [];
                this.headerIndex = [];
                this.detailedHeader = [];
                for (let i = 0; i < headerArray.length; i++) {
                    if (this.presentedFields.indexOf(headerArray[i]) > -1) {
                        let header = {
                            text: headerArray[i],
                            align: 'center',
                            sortable: false,
                            filterable: false,
                            value: headerArray[i]
                        };
                        if (headerArray[i] == 'DO') {
                            header.filterable = true;
                        }
                        this.headers.push(header);
                        this.headerIndex.push(i);
                    }
                    this.detailedHeader.push(headerArray[i]);
                }
            },
            csvToObject() {
                let csvArray = this.csv;
                this.csv = [];
                this.detailedInfo = [];
                for (let i = 0; i < csvArray.length; i++) {
                    let entry = {};
                    let detail = {};
                    for (let j = 0; j < csvArray[i].length; j++) {
                        if (this.detailedHeader[j] == 'KW' || this.detailedHeader[j] == 'AU') {
                            csvArray[i][j] = this.trimString(csvArray[i][j].toString());
                        }
                        let index = this.headerIndex.indexOf(j);
                        if (index > -1) {
                            entry[this.headers[index].value] = csvArray[i][j];
                        }
                        detail[this.detailedHeader[j]] = csvArray[i][j];
                    }
                    this.csv.push(entry);
                    this.detailedInfo.push(detail);
                }
                this.csv = this.csv.reduce((unique, o) => {
                    if (!unique.some(obj => obj.DO === o.DO)) {
                        unique.push(o);
                    }
                    return unique;
                },[]);
                this.detailedInfo = this.detailedInfo.reduce((unique, o) => {
                    if (!unique.some(obj => obj.DO === o.DO)) {
                        unique.push(o);
                    }
                    return unique;
                },[]);
                for (let i = 0; i < this.csv.length; i++) {
                    this.csv[i]['id'] = i;
                    this.detailedInfo[i]['id'] = i;
                }
            },
            trimString(oldString) {
                let newString = oldString.replace('[', '');
                newString = newString.replace(']', '');
                newString = newString.replace(/"/g, '');
                newString = newString.replace(/'/g, '');
                if (newString == '') {
                    newString = 'None';
                }
                return newString;
            }
        },

        computed: {
            showErrorMessage() {
                return this.fileSelected && !this.isValidFileMimeType;
            },
            disabledSubmitButton() {
                return !this.isValidFileMimeType;
            }
        }
    }
</script>