<docs>
  # `csv`

  A button that allows uploading CSV data. Note: the data isn't editable once it's uploaded, but should be re-uploaded from a CSV file when it needs to be changed.

  ### CSV Arguments

  * **delimiter** - alternate delimiter (defaults to comma, of course)
  * **quote** - alternate quote to use (defaults to one double-quote)
  * **help** - description / helper text for the field

  Note: Certain spreadsheet editors like Google Spreadsheets will use triple-quotes if you use both quotes and commas in your cells. Make sure you account for that by changing the `quote` argument:

  ```yaml
  _has:
    input: csv
    quote: '"""'
  ```

  {% hint style="info" %}

  CSV buttons don't have validation or attached buttons.

  {% endhint %}

  ### CSV Data Format

  CSV inputs will format data as an **array of objects*, where each object corresponds to a row and each object's keys are derived from the CSV's column headers.
</docs>

<template>
  <div class="kiln-csv-upload">
    <ui-fileupload color="accent" :name="name" :label="buttonLabel" accept=".csv" @change="update"></ui-fileupload>
    <div class="ui-textbox__feedback" v-if="args.help">
      <div class="ui-textbox__feedback-text">{{ args.help }}</div>
    </div>
  </div>
</template>

<script>
  import _ from 'lodash';
  import { toObject } from 'csvjson';
  import label from '../lib/utils/label';
  import { UPDATE_FORMDATA } from '../lib/forms/mutationTypes';
  import UiFileupload from 'keen/UiFileupload';

  export default {
    props: ['name', 'data', 'schema', 'args'],
    data() {
      return {
        buttonLabel: label(this.name, this.schema)
      };
    },
    methods: {
      update(files) {
        const file = _.head(files),
          reader = new FileReader(),
          store = this.$store;

        reader.readAsText(file);
        reader.onload = (readEvent) => {
          const csvData = _.get(readEvent, 'target.result'),
            parsed = toObject(csvData, {
              delimiter: this.args.delimiter || ',',
              quote: this.args.quote || '"'
            });

          this.$store.commit(UPDATE_FORMDATA, { path: this.name, data: parsed });
          this.buttonLabel = 'CSV File Uploaded';
        };
        reader.onerror = () => {
          store.dispatch('showSnackbar', `Unable to read ${file.fileName}`);
        };
      }
    },
    components: {
      UiFileupload
    }
  };
</script>
