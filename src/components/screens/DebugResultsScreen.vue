<docs>
  This will display all data that has been entered into the experiment up to this point in a table.
  This is useful for debugging.
  Once you are gaoing live with your experiment, you can use the SubmitResultsScreen instead of this one to submit the data to the server and say thank you to the user.
  </docs>
  
  <template>
    <Screen title="Results" class="debugResults">
      <Slide>
        <button @click="downloadCsv">Download all data as csv</button>
        <table v-if="results.length">
          <thead>
            <tr>
              <th v-for="key in Object.keys(results[0])" :key="key">{{ key }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, i) in results" :key="i">
              <td v-for="(key, j) in Object.keys(results[0])" :key="j">
                {{ String(row[key]) }}
              </td>
            </tr>
          </tbody>
        </table>
      </Slide>
    </Screen>
  </template>
  
  <script>
  import Screen from '../Screen';
  import stringify from 'csv-stringify/lib/sync';
  import Slide from '../Slide';
  
  export default {
    name: 'DebugResultsScreen',
    components: { Slide, Screen },
    props: {},
    data() {
      return {
        results: [],
        csv: ''
      };
    },
    mounted() {
      this.results = this.$magpie.getAllData();
      this.expData = this.$magpie.expData;
      // console.log(this.results);
      // this.csv = stringify(this.results, {
      //   columns: Object.keys(this.results[0]),
      //   header: true
      // });
      // // Create the header for expData and its corresponding row
      // let expDataHeader = Object.keys(this.expData).join(",");
      // let expDataRow = Object.values(this.expData).join(",");
  
      // // Create the header for results and their corresponding rows
      // let resultsHeader = Object.keys(this.results[0]).join(",");
      // let resultsRows = this.results.map(r => Object.values(r).join(",")).join("\n");
  
      // // Combine them into one CSV string
      // this.csv = `${expDataHeader}\n${expDataRow}\n\n${resultsHeader}\n${resultsRows}`;
      // Assume this.results is an array of objects
      let resultsKeys = Object.keys(this.results[0]); // Get keys from the first item for consistency
      let resultsRows = this.results.map(result => {
          return resultsKeys.map(key => result[key] || '').join(',');
      }).join("\n");
      let expDataHeader = Object.keys(this.expData).join(",");
      let expDataRow = Object.values(this.expData).join(",");
  
      // Combine with the results header and rows
      let resultsHeader = resultsKeys.join(",");
      this.csv = `${expDataHeader}\n${expDataRow}\n\n${resultsHeader}\n${resultsRows}`;
  
    },
    methods: {
      downloadCsv() {
        let blob = new Blob([this.csv], {
          type: 'text/plain',
          endings: 'native'
        });
        this.download(
          'magpie-' +
            this.$magpie.id +
            '-' +
            new Date().toISOString().slice(0, 10) +
            '.csv',
          blob
        );
      },
  
      download(filename, blob) {
        const element = document.createElement('a');
  
        let objectUrl = URL.createObjectURL(blob);
        element.setAttribute('href', objectUrl);
        element.setAttribute('download', filename);
  
        element.style.display = 'none';
        document.body.appendChild(element);
  
        element.click();
  
        URL.revokeObjectURL(objectUrl);
        document.body.removeChild(element);
      }
    }
  };
  </script>
  <style scoped>
  td {
    max-width: 150px;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  
  .debugResults {
    overflow-x: scroll;
  }
  </style>
  