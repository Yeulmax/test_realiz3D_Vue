<template lang="html">
  <div>
    <div class="ui fixed borderless huge menu">
      <h1>{{ navigation }}</h1>
    </div>
    <br>
    <sui-grid celled>
      <sui-grid-row>
        <sui-grid-column :width="5">
          <sui-label attached="top left">FILTRES</sui-label>
            <sui-dropdown
                class="test"
                fluid
                :options="parentList"
                placeholder="Groupe Parent"
                selection
                multiple
                v-model="parentIdSelected"
            /><br>
          <sui-dropdown
              fluid
              :options="groupTypeList"
              placeholder="Type"
              selection
              multiple
              v-model="groupTypeIdSelected"
          />
        </sui-grid-column>

        <sui-grid-column :width="5">
          <sui-label attached="top left">GROUPES</sui-label>
          <sui-list link>
            <a
                style="color: #000000"
                onMouseOver="this.style.color='#008C59'"
                onMouseOut="this.style.color='#000000'"
                class="groupList"
                is="sui-list-item"
                v-for="group of groupsFiltered"
                :key="group.id"
                v-on:click.prevent="filterLotsByGroupId(group.id)">
              <i class="caret right icon"></i>{{ group.id }} : {{ group.name }}
            </a>
          </sui-list>
        </sui-grid-column>
      </sui-grid-row>

      <sui-grid-row v-if="groupSelected">
        <sui-grid-column>
          <sui-label attached="top left">LOTS</sui-label>
          <div v-if="!lotsExist" class="noLots">Aucun lots !</div>
          <sui-table celled padded v-else>
            <sui-table-header>
              <sui-table-row class="lotHeader">
                <sui-table-header-cell>ID</sui-table-header-cell>
                <sui-table-header-cell>Nom</sui-table-header-cell>
                <sui-table-header-cell>Groupe ID</sui-table-header-cell>
                <sui-table-header-cell collapsing>Actions</sui-table-header-cell>
              </sui-table-row>
            </sui-table-header>
            <sui-table-body>
              <sui-table-row v-for="lot of lotsFiltered" :key="lot.id" class="lotRow">
                <sui-table-cell>{{ lot.id }}</sui-table-cell>
                <sui-table-cell>{{ lot.name }}</sui-table-cell>
                <sui-table-cell>{{ lot.group_id }}</sui-table-cell>
                <sui-table-cell style="text-align: center"><a @click.prevent.stop="deleteLots(lot.id)">&#10060;</a></sui-table-cell>
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
      navigation: 'Accueil',
      groupTypes: [],
      groups: [],
      lots: [],
      groupSelected: null,
      groupsFiltered: [],
      groupsFilteredCache: [],
      lotsFiltered: [],
      lotsExist: false,
      parentIdSelected: [],
      parentList: [],
      groupTypeIdSelected: [],
      groupTypeList: []
    };
  },
  methods: {

    //Retourne les groupes correspondant à un ID parent
    filterGroupsByParentId(id){
      return this.groups.filter(x => x.parent_group_id === id);
    },

    //Retourne les groupes correspondant à un ID de type
    filterGroupsByTypeId(id){
      return this.groups.filter(x => x.group_type_id === id);
    },

    //Filtre les groupes retenu par le filtre parent (présélectionné)
    filterSelectedGroupsByTypeId(id){
      return this.groupsFilteredCache.filter(x => x.group_type_id === id);
    },

    //Filtre les lots correspondant à un ID de groupe
    filterLotsByGroupId(id){
      this.groupSelected = id
      this.lotsFiltered  = this.lots.filter(x => x.group_id === id)
      this.lotsExist     = this.lotsFiltered.length !== 0
    },

    //Applique les filtres selectionné par le menu dropdown
    // Croise les données si nécessaire
    filterGroupSelectedByType(){
      this.groupSelected = null

      if (this.groupTypeIdSelected.length === 0){
        this.groupsFiltered = this.groupsFilteredCache
      }else{
        if (this.parentIdSelected.length === 0){
          this.groupsFiltered = []
          for (let i = 0; i < this.groupTypeIdSelected.length; i++) {
            for (const obj of this.filterGroupsByTypeId(this.groupTypeIdSelected[i])) {
              this.groupsFiltered.push(obj)
            }
          }
        }else{
          let tempGroupsFiltered = []
          for (let i = 0; i < this.groupTypeIdSelected.length; i++) {
            for (const obj of this.filterSelectedGroupsByTypeId(this.groupTypeIdSelected[i])) {
              tempGroupsFiltered.push(obj)
            }
          }
          this.groupsFiltered = tempGroupsFiltered
        }
      }

    },

    //Supprime un lot
    async deleteLots(lotId){
      try {
        await axios.delete(`http://localhost/test_realiz3D/public/api/lots/` + lotId);
      } catch (e) {
        console.error(e);
      }

      await this.updateLots()
      this.filterLotsByGroupId(this.groupSelected)
    },

    //Met à jour la liste des lots
    async updateLots(){
      try {
        const result = await axios.get(`http://localhost/test_realiz3D/public/api/lots`);
        this.lots = result.data.reverse();
      } catch (e) {
        console.error(e);
      }
    },

    //Vérifie sur un groupe possède un parent
    hasParent(groupId){
      return !!this.groups.filter(x => x.id === groupId)[0].parent_group_id;
    },

    //Renvoie le nom du groupe parent
    getParentName(groupId){
      let parentId = this.groups.filter(x => x.id === groupId)[0].parent_group_id
      return this.groups[parentId - 1].name
    },

    //Met à jour l'indicateur de navigation
    updateNavigation(){
      let groupId = this.groupSelected

      if (groupId){
        if (this.hasParent(groupId)){
          this.navigation = this.getParentName(groupId) + " -> " + this.groups[groupId - 1].name
        }else{
          this.navigation = this.groups[groupId - 1].name
        }
      }else{
        this.navigation = 'Accueil'
      }
    },

    //Crée la liste des groupes parents
    // en fonction du contenu de la BDD (Dynamique)
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
    },

    //Crée la liste des groupes en fonction du
    // contenu de la BDD (Dynamique)
    setGroupTypeList(){
      for (const groupType of this.groupTypes){
        let newElement = {text: groupType.id.toString() + ' - ' + groupType.label, value: groupType.id }
        this.groupTypeList.push(newElement);
      }
    }
  },
  watch: {
    groupTypeIdSelected(){
      this.filterGroupSelectedByType()
    },

    groupSelected(){
      this.updateNavigation()
    },

    parentIdSelected(){
      this.groupSelected = null
      this.groupsFiltered = []

      for (let i = 0; i < this.parentIdSelected.length; i++) {
        for (const obj of this.filterGroupsByParentId(this.parentIdSelected[i])) {
          this.groupsFiltered.push(obj)
        }
      }
      this.groupsFilteredCache = this.groupsFiltered
      this.filterGroupSelectedByType()
    },

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
    await this.updateLots()

    //Initialise le contenu des menus dropdown
    this.setParentList();
    this.setGroupTypeList();

    //Initialise la sélection de groupe parent
    this.parentIdSelected.push(null)
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
  background-color: #008C59;
  padding: 0.5em 1em;
}

.groupList{
  font-weight: bold;
  font-size: x-large;
}

.lotHeader{
  font-weight: bold;
  font-size: large;
}

.lotRow{
  font-size: large;
}

.ui.celled.grid{
  margin-top: 100px;
}
</style>
