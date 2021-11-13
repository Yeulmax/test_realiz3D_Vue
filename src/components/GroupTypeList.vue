<template lang="html">
  <div>
    <div class="ui fixed borderless huge menu">
      <h1>Accueil</h1>
    </div>
    <br><br><br><br><br><br>
    <sui-dropdown
        placeholder="GroupType"
        selection
        :options="options"
        v-model="current"
    />
    <sui-grid celled>
      <sui-grid-row>
        <sui-grid-column :width="5">
          <sui-label attached="top left">GROUPTYPE</sui-label>
            <sui-dropdown
                fluid
                multiple
                :options="parentId"
                placeholder="Parent ID"
                selection
                v-model="parentIdSelected"
            /><br>
            <sui-dropdown
                fluid
                multiple
                :options="parentId"
                placeholder="Type"
                selection
                v-model="parentIdSelected"
            />
        </sui-grid-column>
        <sui-grid-column :width="5">
          <sui-label attached="top left">GROUPS</sui-label>
          <ul>
            <li v-for="group of groupsFilterByParentId(current)" :key="group.id" v-on:click="groupSelected = group.id">
              {{ group.id }} - {{ group.name }}
            </li>
          </ul>
        </sui-grid-column>
      </sui-grid-row>
      <sui-grid-row v-if="groupSelected">
        <sui-grid-column >
          <sui-label attached="top left">LOTS</sui-label>
          <div v-if="!lotsExist">Pas de lots</div>
          <ul>
            <li v-for="lot of lotsFilterByGroupId(groupSelected)" :key="lot.id">
              {{ lot.id }} - {{ lot.name }}
            </li>
          </ul>
        </sui-grid-column>
      </sui-grid-row>
    </sui-grid>

    <sui-table celled padded>
      <sui-table-header>
        <sui-table-row>
          <sui-table-header-cell collapsing>ID</sui-table-header-cell>
          <sui-table-header-cell>Nom</sui-table-header-cell>
          <sui-table-header-cell>Group</sui-table-header-cell>
        </sui-table-row>
      </sui-table-header>

      <sui-table-body>
        <sui-table-row>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
        </sui-table-row>
        <sui-table-row>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
        </sui-table-row>
        <sui-table-row>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
          <sui-table-cell>Cell</sui-table-cell>
        </sui-table-row>
      </sui-table-body>


      <sui-table-footer>
        <sui-table-row>
          <sui-table-header-cell colspan="3">
            <sui-menu pagination>
              <a is="sui-menu-item" icon>
                <sui-icon name="left chevron" />
              </a>
              <a is="sui-menu-item">1</a>
              <a is="sui-menu-item">2</a>
              <a is="sui-menu-item">3</a>
              <a is="sui-menu-item">4</a>
              <a is="sui-menu-item" icon>
                <sui-icon name="right chevron" />
              </a>
            </sui-menu>
          </sui-table-header-cell>
        </sui-table-row>
      </sui-table-footer>
    </sui-table>

  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "GroupTypeList",
  data() {
    return {
      test: null,
      groupSelected: null,
      groupsParentSelected: null,
      groupTypes: [],
      groups: [],
      lots: [],
      lotsExist: false,
      parentIdSelected: [],
      parentId: [
        { key: 'angular', text: 'Angular', value: 'angular' },
        { key: 'css', text: 'CSS', value: 'css' },
        { key: 'design', text: 'Graphic Design', value: 'design' },
      ],
      current: null,
      counter: 0,
      options: [
        {
          text: 'NULL',
          value: null,
        },
        {
          text: '1',
          value: 1,
        },
      ],
    };
  },
  computed: {
    parentIdList(){
      return true
    }
  },
  methods: {
    groupsFilterByParentId(id){
      return this.groups.filter(x => x.parent_group_id === id);
    },
    lotsFilterByGroupId(id){
      let lotsFiltered = this.lots.filter(x => x.group_id === id)
      this.lotsExist = lotsFiltered.length !== 0;
      return lotsFiltered;
    }
  },
  async created() {
    try {
      let result = await axios.get(`http://localhost/test_realiz3D/public/api/groupTypes`);
      this.groupTypes = result.data.reverse();
    } catch (e) {
      console.error(e);
    }

    try {
      const result = await axios.get(`http://localhost/test_realiz3D/public/api/groups`);
      this.groups = result.data.reverse();
    } catch (e) {
      console.error(e);
    }

    try {
      const result = await axios.get(`http://localhost/test_realiz3D/public/api/lots`);
      this.lots = result.data.reverse();
    } catch (e) {
      console.error(e);
    }
  }
};
</script>

<style scoped>
h1 {
  padding-top: 5px;
  padding-left: 50px;
  color: #ffffff;
  font-size: 2em;
}

ul {
  margin-top: 20px;
}

li{
  font-weight: bold;
  font-size: large;
}

body > .ui.container {
  margin-top: 6em;
}

.ui.fixed.borderless.menu {
  background-color: #4a008a;
  padding: 0.5em 1em;
}

</style>
