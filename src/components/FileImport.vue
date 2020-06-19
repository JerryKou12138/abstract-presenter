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
            <button type="submit" :disabled="disabledSubmitButton" class="btn btn-primary" @click.prevent="load">
                submit
            </button>
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
        },

        data: () => ({
            hasHeaders: true,
            headers: null,
            csv: null,
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
                    this.csvToObject();
                    this.headerToObject();
                    _this.$emit('input', _this.csv);
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
                this.headers = [
                    {
                        text: 'Abstract Summary',
                        align: 'start',
                        sortable: false,
                        value: headerArray[0]
                    }
                ];
                for (let i = 1; i < headerArray.length; i++) {
                    this.headers.push(
                        {
                            text: headerArray[i],
                            value: headerArray[i]
                        }
                    );
                }
            },
            csvToObject() {
                let csvArray = this.csv;
                let headerSize = this.headers.length;
                this.csv = [];
                for (let i = 0; i < csvArray.length; i++) {
                    let entry = {};
                    for (let j = 0; j < headerSize; j++) {
                        entry[this.headers[j]] = csvArray[i][j];
                    }
                    this.csv.push(entry);
                }
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