<template >
  <div>
    <div class="country-filter">
      <label>Country Filter : </label>
      <select v-model="countryFilter">
        <option value="ALL">All</option>
        <option
          :selected="onFilterSelect()"
          v-for="(option, index) in countryList"
          :key="index"
          :value="option"
        >
          {{ option }}
        </option>
      </select>
    </div>
    <div class="table-container">
      <table class="table">
        <thead>
          <tr>
            <th>Key</th>

            <th>Country</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(objData, keyIndex) in filteredObjData()" :key="keyIndex">
            <template v-for="(subObj, subIndex) in objData" :key="subIndex">
              <td>{{ subIndex }}</td>
              <tr v-for="(obj, index) in subObj" :key="index">
                <td>{{ index }}</td>
                <div
                  style="display: inline-block"
                  v-for="(miniObj, miniIndex) in obj"
                  :key="miniIndex"
                >
                  <td>
                    <label style="text-transform: capitalize"
                      >{{ miniIndex }}:</label
                    >
                    <input
                      :class="
                        editedKeyList
                          .toString()
                          .includes([subIndex, index].toString())
                          ? 'edited-input'
                          : ''
                      "
                      :disabled="
                        !(keyIndex == currentKeyIndex && index == currentIndex)
                      "
                      v-model="obj[miniIndex]"
                    />
                  </td>
                </div>
                <td>
                  <button
                    v-if="
                      !(keyIndex == currentKeyIndex && index == currentIndex)
                    "
                    @click="editData(keyIndex, subIndex, index, objData)"
                  >
                    Edit
                  </button>
                  <button
                    v-if="keyIndex == currentKeyIndex && index == currentIndex"
                    @click="editedData(keyIndex, subIndex, index, objData)"
                  >
                    OK
                  </button>
                </td>
                <td>
                  <button
                    v-if="keyIndex == currentKeyIndex && index == currentIndex"
                    @click="cancelEditData(keyIndex, index)"
                  >
                    Cancel
                  </button>
                </td>
              </tr>
            </template>
          </tr>
        </tbody>
      </table>
      <div style="display: inline-block">
        <button class="save-button" @click="saveData()">Save</button>
      </div>
      <div style="display: inline-block">
        <button class="save-button" @click="resetData()">Reset</button>
      </div>
      <div class="pagination">
        <pagination
          v-model="page"
          :records="metaObjectListData.length * totalRecordMultiplier"
          :per-page="perPageData"
          @paginate="paginatedData($event)"
        />
      </div>
    </div>
  </div>
</template>


<script>
import Pagination from "v-pagination-3";
export default {
  components: { Pagination },
  props: {
    metaObject: {},
  }, //props can be array or object and mutating a prop locally is considered an anti-pattern
  data() {
    return {
      metaObjectListData: [],
      currentKeyIndex: null,
      currentIndex: null,
      page: 1,
      perPageData: 10,
      countryList: [],
      countryFilter: "ALL",
      dataFiltered: false,
      tempMetaObjectListData: [],
      totalRecordMultiplier: null,
      compomentMetaObject: {},
      editedDataList: [],
      editedKeyList: [],
    };
  },
  methods: {
    editData(keyIndex, subIndex, index, objData) {
      this.currentKeyIndex = keyIndex;
      this.currentIndex = index;
      console.log(keyIndex, subIndex, index, objData);
    },
    editedData(keyIndex, subIndex, index, objData) {
      console.log(keyIndex, subIndex, index, objData);
      this.editedDataList.push([subIndex, index, objData]);
      this.editedKeyList.push([subIndex, index]);
      console.log(
        "this.editedKeyList :",
        this.editedKeyList.toString().includes(["key1", "UK"].toString())
      );
      console.log(this.editedKeyList.indexOf(["key1", "UK"]));
      this.metaObjectListData.forEach((data, i) => {
        for (let key in data) {
          if (key == subIndex) {
            this.metaObjectListData[i][key][index] = objData[subIndex][index];
          }
        }
      });

      this.currentKeyIndex = null;
      this.currentIndex = null;
      // console.log("this.metaObjectListData :", this.metaObjectListData);
      // console.log("this.compomentMetaObject :", this.compomentMetaObject);
    },
    cancelEditData() {
      this.currentKeyIndex = null;
      this.currentIndex = null;
    },
    saveData() {
      if (this.editedDataList.length != 0) {
        this.editedDataList.forEach((dataArray) => {
          this.saveEachData(dataArray[0], dataArray[1], dataArray[2]);
        });
        this.editedDataList = [];
        this.editedKeyList = [];
        alert("Data Saved !!");
      } else {
        alert("Please edit data first !!");
      }
      // console.log("this.compomentMetaObject :", this.compomentMetaObject);
    },
    saveEachData(subIndex, index, obj) {
      this.currentKeyIndex = null;
      this.currentIndex = null;
      this.compomentMetaObject[subIndex][index] = obj[subIndex][index];
    },
    resetData() {
      this.metaObjectListData = this.convertObjTOArray(
        JSON.parse(JSON.stringify(this.compomentMetaObject))
      );
      this.editedDataList = [];
      this.editedKeyList = [];
    },
    paginatedData(data) {
      console.log("Current Page :", data, this.page);
    },
    filteredObjData() {
      // console.log("Filtering Data");
      let tempObjArray = JSON.parse(JSON.stringify(this.metaObjectListData));
      tempObjArray.filter((data, index) => {
        for (let key in data) {
          for (let country in data[key]) {
            if (this.countryFilter == "ALL") {
              return true;
            } else if (this.countryFilter != country) {
              if (tempObjArray[index][key][country]) {
                delete tempObjArray[index][key][country];
              }
            }
          }
        }
      });
      if (this.countryFilter == "ALL") {
        this.totalRecordMultiplier = this.countryList.length;
        return tempObjArray.slice(2 * (this.page - 1), 2 * this.page);
      } else {
        this.totalRecordMultiplier = 1;
        if (this.page > this.metaObjectListData.length / this.perPageData) {
          this.page = 1;
        }
        return tempObjArray.slice(10 * (this.page - 1), 10 * this.page);
      }
    },
    onFilterSelect() {
      // console.log("filter selected");
      this.dataFiltered = true;
    },
    convertObjTOArray(obj) {
      let resultArray = [];
      let tempObj;
      if (obj instanceof Object) {
        for (let key in obj) {
          tempObj = {};
          tempObj[key] = obj[key];
          resultArray.push(tempObj);
        }
        return resultArray;
      } else {
        return resultArray;
      }
    },
  },
  // watch: {
  //   metaObject: {
  //     immediate: true,
  //     handler() {
  //       this.compomentMetaObject = JSON.parse(JSON.stringify(this.metaObject));
  //        console.log("Inside Watch metaObject : ", this.compomentMetaObject);
  //     },
  //   },
  // },
  created: function () {
    console.log("CountryTable COMPONENT CREATED");
  },
  mounted: function () {
    console.log("CountryTable COMPONENT MOUNTED");
    this.compomentMetaObject = JSON.parse(JSON.stringify(this.metaObject));
    this.metaObjectListData = this.convertObjTOArray(
      JSON.parse(JSON.stringify(this.metaObject))
    );
    for (let data in this.metaObjectListData[0]["key1"]) {
      this.countryList.push(data);
    }
  },
};
</script>


<style scoped>
.display-none {
  display: none;
}

.table {
  margin: 10px;
  border: 1px solid black;
  margin-top: 3%;
}

.pagination {
  width: 100%;
  padding: 10px;
}

tr {
  border-top: 1px solid black;
  border-bottom: 1px solid black;
  border-left: 1px solid black;
}

.country-filter {
  padding-top: 2%;
}

.edited-input {
  border: 2px solid green;
}

.save-button {
  margin-bottom: 15px;
  margin-left: 10px;
  padding: 8px 20px;
  font-size: 16px;
}
</style>

