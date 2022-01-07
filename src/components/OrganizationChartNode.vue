<template>
  <table>
    <tbody>
      <tr>
       <td :colspan="datasource.children && datasource.children.length ? datasource.children.length*2: null">
        <div class="node" :id="datasource.id" @click.stop="handleClick(datasource)">
          <slot :node-data="datasource">
            <div class="title">
              <!-- <i class="fa fa-users symbol"></i> -->
              <b> <label style="margin-bottom: 0;">{{ datasource.nombre }}</label></b> <br />
              <label style="margin-bottom: 0;">{{ datasource.puesto }}</label>
            </div>
            <div class="content">
              <img class="profile-picture" :src="datasource.img" alt="" height="50">
            </div>
          </slot>
          <div class="popup" :id="`popup-${datasource.id}`">
            <div style="display: flex; justify-content: space-around; align-items: center;flex-direction: column;">
              <div style="display: flex;justify-content: center;width: 100%;margin-left: 10px;margin-top: 10px;">
                <label class="lbl-name">{{ datasource.nombre }}</label>
                <label>{{ datasource.plaza || 'Ninguna' }}</label>
              </div>
              <div style="display: flex;width: 100%;">
                <img class="profile-picture" :src="datasource.img" alt="" height="90" style="margin-top: 5px;margin-left: 5px;">
                <div style="display: flex;justify-content: space-around;align-items: flex-start;flex-direction: column;margin-left: 5px;">
                  <label class="lbl-field">Departamento:</label>
                  <label class="lbl-value">{{ datasource.departamento || 'Ninguno' }}</label>
                  <label class="lbl-field">Sucursal:</label>
                  <label class="lbl-value">{{ datasource.sucursal || 'Ninguna' }}</label>
                  <label class="lbl-field">Puesto:</label>
                  <label class="lbl-value">{{ datasource.puesto || 'Ninguno' }}</label>
                </div>
              </div>
              <div style="width: 100%;
                      padding: 5px;
                      display: flex;
                      margin-left: 5px;">
                <label>Plaza: </label>
                <label>{{ datasource.plaza || 'Ninguna' }}</label>
              </div>
            </div><!--This div allows for CCW rotation of number in the CW rotated popup-->
          </div>
        </div>
       </td>
      </tr>
      <template v-if="datasource.children && datasource.children.length">
        <tr class="lines">
          <td :colspan="datasource.children.length*2">
            <div class="downLine"></div>
          </td>
        </tr>
        <tr class="lines">
          <td class="rightLine"></td>
         <template v-for="n in (datasource.children.length-1)">
            <td class="leftLine topLine" :key="n"></td>
            <td class="rightLine topLine" :key="n"></td>
         </template>
          <td class="leftLine"></td>
        </tr>
        <tr class="nodes">
          <td colspan="2" v-for="child in datasource.children" :key="`${child.id.toString()}-${datasource.id.toString()}`">
            <node :datasource="child" :handle-click="handleClick">
              <template v-for="slot in Object.keys($scopedSlots)" :slot="slot" slot-scope="scope">
                <slot :name="slot" v-bind="scope"/>
              </template>
            </node>
          </td>
        </tr>
      </template>
    </tbody>
  </table>
</template>

<script>
export default {
  name: 'node',
  props: {
    nodeId: String,
    datasource: Object,
    handleClick: Function
  },
  methods: {
  }
};
</script>

<style>
</style>
