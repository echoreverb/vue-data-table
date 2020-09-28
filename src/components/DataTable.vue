<template>
  <div class="table" v-if="shownRecords.length" ref="start">
    <form class="table__searchbar" @submit.prevent="submitSearch(search)">
      <label for="search" class="table__input-label"
        ><input
          v-model="search"
          placeholder="Введите поисковый запрос"
          class="table__input"
          type="text"
          required
          spellcheck="false"
      /></label>
      <button class="table__searchbar-button">Найти</button>
      <button type="button" class="table__searchbar-button" @click="resetSearch">Сбросить</button>
    </form>
    <p v-if="error" class="table__error">{{ error }}</p>
    <table :class="['table__container', { table__container_invisible: error }]">
      <thead>
        <tr class="table__header-row">
          <th
            v-for="columnHeading in columnHeadings"
            :key="columnHeadings.indexOf(columnHeading)"
            :class="['table__header-cell', { 'table__header-cell_sorted': columnHeading.isSorted }]"
            @click="handleSort(columnHeading)"
            class="table__header-cell"
          >
            {{ columnHeading.name }}
            <span v-if="!columnHeading.isSorted || columnHeading.sortedUp" class="table__sort-icon"
              >&uarr;</span
            >
            <span v-else class="table__sort-icon">&darr;</span>
          </th>
        </tr>
      </thead>
      <tbody v-if="!error">
        <tr
          v-for="shownRecord in shownRecords"
          :key="shownRecord.id"
          @click="$emit('record-click', shownRecord)"
          class="table__body-row"
        >
          <td
            v-for="columnHeading in columnHeadings"
            class="table__body-cell"
            :key="`${columnHeadings.indexOf(columnHeading)}x${shownRecord.id}`"
          >
            {{ shownRecord[columnHeading.dataProp] }}
          </td>
        </tr>
      </tbody>
      <tr class="table__body-row table__body-row_invisible">
        <td
          v-for="longestString in longestStrings"
          class="table__body-cell table__body-cell_invisible"
          :key="'ghost' + longestStrings.indexOf(longestString)"
        >
          {{ longestString + compensateString }}
        </td>
      </tr>
    </table>

    <div v-if="!error" class="table__pagination">
      <button
        :class="[
          'table__pagination-element',
          'table__pagination-element_type_arrow',
          { 'table__pagination-element_disabled': isFirstPage },
        ]"
        @click="toFirstPage"
      >
        &laquo;
      </button>
      <button
        :class="[
          'table__pagination-element',
          'table__pagination-element_type_arrow',
          { 'table__pagination-element_disabled': isFirstPage },
        ]"
        @click="prevPage"
      >
        &lsaquo;
      </button>
      <button class="table__pagination-element">
        {{ `${indexFrom + 1} - ${indexTo} из ${records.length}` }}
      </button>
      <button
        :class="[
          'table__pagination-element',
          'table__pagination-element_type_arrow',
          { 'table__pagination-element_disabled': isLastPage },
        ]"
        @click="toNextPage"
      >
        &rsaquo;
      </button>
      <button
        :class="[
          'table__pagination-element',
          'table__pagination-element_type_arrow',
          { 'table__pagination-element_disabled': isLastPage },
        ]"
        @click="toLastPage"
      >
        &raquo;
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    recordsProp: Array,
    columnsProp: Array,
  },
  data() {
    return {
      records: this.recordsProp,
      recordsBuffer: [],
      columnHeadings: this.columnsProp,
      longestStrings: [],
      recordsPerPage: 15,
      indexFrom: 0,
      search: '',
      error: '',
      compensateString: '___',
    };
  },
  computed: {
    shownRecords() {
      return this.records.slice(this.indexFrom, this.indexFrom + this.recordsPerPage);
    },
    isLastPage() {
      if (this.indexFrom + this.recordsPerPage >= this.records.length) {
        return true;
      } else {
        return false;
      }
    },
    isFirstPage() {
      if (this.indexFrom === 0) {
        return true;
      } else {
        return false;
      }
    },
    indexTo() {
      const index = this.indexFrom + this.recordsPerPage;
      if (index > this.records.length) {
        return this.records.length;
      }
      return index;
    },
  },
  created() {
    const mappedRecords = this.records.map(record => this.flattenObject(record));
    this.records = mappedRecords;
    this.recordsBuffer = mappedRecords;

    this.longestStrings = this.columnHeadings.map(column => {
      return this.recordsBuffer.reduce((longest, record) => {
        return record[column.dataProp].length > longest.length ? record[column.dataProp] : longest;
      }, '');
    });

    this.columnHeadings.map(object => {
      object.isSorted = false;
      object.sortedUp = false;
    });
  },
  methods: {
    handleToStartClick() {
      this.indexFrom = 0;
    },
    toNextPage() {
      if (!this.isLastPage) {
        this.indexFrom += this.recordsPerPage;
      }
    },
    prevPage() {
      if (!this.isFirstPage) {
        this.indexFrom -= this.recordsPerPage;
      }
    },
    toFirstPage() {
      this.indexFrom = 0;
    },
    toLastPage() {
      this.indexFrom =
        this.records.length % this.recordsPerPage
          ? Math.floor(this.records.length / this.recordsPerPage) * this.recordsPerPage
          : this.records.length - this.recordsPerPage;
    },
    submitSearch(value) {
      this.error = '';
      const toFilter = this.recordsBuffer;
      const filtered = toFilter.filter(record => {
        const filterValue = String(value);
        let bingo = false;
        for (let prop in record) {
          const currentValue = String(record[prop]);
          if (currentValue.includes(filterValue)) {
            bingo = true;
            break;
          }
        }
        return bingo;
      });
      if (!filtered.length) {
        this.showNoResults();
        return;
      }
      this.records = filtered;
    },
    showNoResults() {
      this.error = 'По данному запросу ничего не найдено';
    },
    resetSearch() {
      this.search = '';
      this.error = '';
      this.records = this.recordsBuffer;
    },
    setIndexFrom(index) {
      this.indexFrom = (index - 1) * this.recordsPerPage;
      this.$refs.start.scrollIntoView({
        behavior: 'smooth',
      });
    },
    resetIndexFrom() {
      this.indexFrom = 0;
    },
    diveObject(currentKey, into, target) {
      for (let i in into) {
        let newKey = i;
        const newVal = into[i];
        if (currentKey.length > 0) {
          newKey = currentKey + '.' + i;
        }
        if (typeof newVal === 'object') {
          this.diveObject(newKey, newVal, target);
        } else {
          target[newKey] = newVal;
        }
      }
    },
    flattenObject(nestedObj) {
      const newObj = {};
      this.diveObject('', nestedObj, newObj);
      return newObj;
    },
    handleSort(columnHeading) {
      this.toFirstPage();
      this.resetIndexFrom();
      const prop = columnHeading.dataProp;
      const factor = columnHeading.sortedUp ? -1 : 1;
      this.records.sort((a, b) => {
        let first = a[prop];
        let second = b[prop];
        if (typeof first === 'string') {
          first = first.toLowerCase();
          second = second.toLowerCase();
        }
        if (first > second) {
          return factor;
        }
        if (first < second) {
          return -1 * factor;
        }
        return 0;
      });
      this.columnHeadings.forEach(obj => {
        if (obj.dataProp === prop) {
          obj.sortedUp = !obj.sortedUp;
          obj.isSorted = true;
        } else {
          obj.isSorted = false;
          obj.sortedUp = false;
        }
      });
    },
  },
};
</script>

<style scoped>
.table {
  box-sizing: border-box;
  width: fit-content;
  margin: 0 auto;
  padding: 30px 0;
}

.table__searchbar {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  font-weight: normal;
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
  width: 100%;
  margin-bottom: 20px;
  justify-content: space-between;
}

.table__searchbar-button {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  font-weight: normal;
  width: 120px;
  margin-left: 10px;
  border: none;
  outline: none;
  background-color: #536ffc;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  color: #fff;
  cursor: pointer;
}

.table__input-label {
  box-sizing: border-box;
  display: block;
  width: calc(100% - 260px);
}
.table__error {
  display: block;
  min-width: 100px;
  min-height: 100px;
  padding: 20px;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
}

.table__input {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  font-weight: normal;
  padding: 5px 10px;
  box-sizing: border-box;
  border: none;
  width: 100%;
  height: 37px;
  outline: none;
  border-bottom: 1px solid #ececec;
}

.table__input:focus {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.table__searchbar-button:hover {
  background-color: #5d77fe;
}

.table__searchbar-button:active {
  background-color: #3d5dfc;
}

.table__container {
  border-collapse: collapse;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  font-weight: normal;
  min-width: 400px;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  margin: auto;
  visibility: visible;
}

.table__container.table__container_invisible {
  visibility: hidden;
}

.table__header-row {
  background-color: #536ffc;
  color: #fff;
}

.table__sort-icon {
  visibility: hidden;
}

.table__header-cell {
  cursor: pointer;
}

.table__header-cell:hover .table__sort-icon {
  visibility: visible;
  opacity: 0.6;
}

.table__header-cell.table__header-cell_sorted > .table__sort-icon {
  visibility: visible;
  opacity: 1;
}

.table__header-cell {
  padding: 16px;
  padding-left: 32px;
  text-align: left;
}

.table__body-row {
  border-bottom: 1px solid #d9dfff;
  cursor: pointer;
}

.table__body-cell {
  max-width: 350px;
  word-break: normal;
  padding: 12px;
  padding-left: 32px;
  text-align: left;
}

.table__body-cell.table__body-cell_invisible {
  height: 0;
  padding: 0;
  padding-left: 32px;
  line-height: 0;
  visibility: hidden;
}

.table__body-row_invisible {
  height: 0;
  /* visibility: hidden; */
}

.table__body-row:last-of-type {
  /* border-bottom: 1px solid #536ffc; */
  border-bottom: none;
}

.table__body-row:hover {
  background-color: #f2f3f7;
}

.table__body-row:active {
  background-color: #e3e6ef;
}

.table__pagination {
  display: flex;
  margin: 20px auto;
  width: fit-content;
}

.table__pagination-element {
  min-width: 160px;
  margin-right: 10px;
  display: block;
  padding: 7px 10px;
  background-color: #fff;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
  font-weight: normal;
  outline: none;
  border: none;
}

.table__pagination-element.table__pagination-element_type_arrow {
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  min-width: 34px;
  font-size: 18px;
  padding: 0 10px 2px;
}

.table__pagination-element.table__pagination-element_disabled {
  opacity: 0.6;
  box-shadow: none;
  cursor: default;
}

.table__pagination-element:last-child {
  margin-right: 0;
}

/* .table__header-cell:nth-child(1) {
  min-width: 200px;
}

.table__header-cell:nth-child(2) {
  min-width: 150px;
}

.table__header-cell:nth-child(3) {
  min-width: 210px;
}

.table__header-cell:nth-child(4) {
  min-width: 250px;
}

.table__header-cell:nth-child(5) {
  min-width: 70px;
} */
</style>
