<template lang="html">
  <div>
    <div class="ui fixed borderless huge menu">
      <h1>Accueil</h1>
    </div>
    <br><br><br><br><br><br>
    <sui-grid celled>
      <sui-grid-row>
        <sui-grid-column :width="5">
          <sui-label attached="top left">FILTRES</sui-label>
            <sui-dropdown
                fluid
                :options="parentList"
                placeholder="Groupe Parent"
                selection
                v-model="parentIdSelected"
            /><br>
          <sui-dropdown
              fluid
              :options="options"
              placeholder="Type"
              selection
              v-model="testSelected"
              v-on:submit.stop
          />
        </sui-grid-column>
        <sui-grid-column :width="5">
          <sui-label attached="top left">GROUPES</sui-label>

          <sui-list link>
            <a
                style="color: #000000"
                onMouseOver="this.style.color='#5c5c5c'"
                onMouseOut="this.style.color='#000000'"
                class="groupList"
                is="sui-list-item"
                v-for="group of groupsFilterByParentId(parentIdSelected)"
                :key="group.id"
                v-on:click.prevent="filterLotsByGroupId(group.id)">
              {{ group.id }} : {{ group.name }}
            </a>
          </sui-list>
        </sui-grid-column>
      </sui-grid-row>

      <sui-grid-row v-if="groupSelected">
        <sui-grid-column >
          <sui-label attached="top left">LOTS</sui-label>
          <div v-if="!lotsExist" class="noLots">Aucun lots !</div>
          <sui-table celled padded v-else>
            <sui-table-header>
              <sui-table-row>
                <sui-table-header-cell collapsing>ID</sui-table-header-cell>
                <sui-table-header-cell>Nom</sui-table-header-cell>
                <sui-table-header-cell>Groupe ID</sui-table-header-cell>
              </sui-table-row>
            </sui-table-header>
            <sui-table-body>
              <sui-table-row v-for="lot of lotsFiltered" :key="lot.id">
                <sui-table-cell>{{ lot.id }}</sui-table-cell>
                <sui-table-cell>{{ lot.name }}</sui-table-cell>
                <sui-table-cell>{{ lot.group_id }}</sui-table-cell>
              </sui-table-row>
            </sui-table-body>
          </sui-table>


        </sui-grid-column>
      </sui-grid-row>
    </sui-grid>



  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "GroupTypeList",
  data() {
    return {
      test: null,
      testSelected: null,
      groupSelected: null,
      groupsParentSelected: null,
      groupTypes: [],
      groups: [],
      lots: [],
      lotsFiltered: [],
      lotsExist: false,
      parentIdSelected: null,
      parentList: [],
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

  },
  methods: {
    testa(){
      //e.stopPropagation()
      console.log('Hey')
    },
    groupsFilterByParentId(id){
      return this.groups.filter(x => x.parent_group_id === id);
    },

    filterLotsByGroupId(id){
      this.groupSelected = id
      this.lotsFiltered  = this.lots.filter(x => x.group_id === id)
      this.lotsExist     = this.lotsFiltered.length !== 0;
    },

    setParentList(){
      //Récupération des ID des groupes parents
      let parentIds = []
      this.groups.forEach(element => parentIds.push(element['parent_group_id']))

      //Suppression des doublons
      parentIds = [...new Set(parentIds)]

      //Création des attributs du menu de sélection
      for(let parentId of parentIds){
        if (parentId === null){
          let newElement = {text: 'NULL', value: null }
          this.parentList.push(newElement);
        }else{
          let nameOfParent = this.groups.find(x => x.id === parentId).name
          let newElement = {text: parentId.toString() + ' - ' + nameOfParent, value: parentId }
          this.parentList.push(newElement);
        }
      }
    }
  },
  watch: {
    parentIdSelected(){
      this.groupSelected = null
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
      this.setParentList();
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
.noLots{
  font-weight: bold;
  font-size: xx-large;
  padding-bottom: 10px;
  margin-left: 50px;
  color: #db0000;
}

body > .ui.container {
  margin-top: 6em;
}

.ui.fixed.borderless.menu {
  background-color: #4a008a;
  padding: 0.5em 1em;
}

.groupList{
  font-weight: bold;
  font-size: x-large;
}

</style>
