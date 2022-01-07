<template>
  <div id="app">
    <org-chart :datasource="ds" @node-click="selectNode">
<!--       <template slot-scope="{ nodeData }">
        <b @click="selectNode(nodeData)">{{ nodeData.name }}</b>
      </template> -->
    </org-chart>
  </div>
</template>

<script>
import OrgChart from './components/OrganizationChartContainer.vue'

export default {
  name: 'app',
  components: {
    OrgChart
  },
  data () {
    return {
      ds: {
        'id': '1',
        'nombre': 'Lao Lao',
        'puesto': 'general manager',
        'img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
        'children': [
          { 'id': '2', 'nombre': 'Bo Miao', 'puesto': 'department manager','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
          { 'id': '3', 'nombre': 'Su Miao', 'puesto': 'department manager','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
            'children': [
              { 'id': '4', 'nombre': 'Tie Hua', 'puesto': 'senior engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
              { 'id': '5', 'nombre': 'Hei Hei', 'puesto': 'senior engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
                'children': [
                  { 'id': '6', 'nombre': 'Pang Pang', 'puesto': 'engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
                  { 'id': '7', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' }
                ]
               }
             ]
           },
          { 'id': '8', 'nombre': 'Hong Miao', 'puesto': 'department manager','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
                'children': [
                  { 'id': '9', 'nombre': 'Pang Pang', 'puesto': 'engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
                  { 'id': '10', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' ,
                'children': [
                  { 'id': '11', 'nombre': 'Pang Pang', 'puesto': 'engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
                  { 'id': '12', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
                'children': [
                  { 'id': '13', 'nombre': 'Pang Pang', 'puesto': 'engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
                  { 'id': '14', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
                'children': [
                  { 'id': '15', 'nombre': 'Pang Pang', 'puesto': 'engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' },
                  { 'id': '16', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg',
                'children': [
                  { 'id': '17', 'nombre': 'Pang Pang', 'puesto': 'engineer' ,'img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg'},
                  { 'id': '18', 'nombre': 'Xiang Xiang', 'puesto': 'UE engineer' ,'img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg'}
                ] }
                ] }
                ] }
                ]}
                ] },
          { 'id': '19', 'nombre': 'Chun Miao', 'puesto': 'department manager','img': 'http://localhost/adagioRH.Web/fotos/empleados/ADG0002.jpg' }
        ]
      }
    }
  },
  methods: {
    selectNode (nodeData) {
      alert('node ' + nodeData.name + ' is selected')
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
