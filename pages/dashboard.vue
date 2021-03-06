<template>
    <div>

        <v-layout row align-center justify-center>
            <v-flex xs12>

                <v-card v-if="numberOfTotalWidgets == 0">
                    <v-alert
                    :value="true"
                    color="warning"
                    icon="new_releases"
                    class="headline"
                    >

                    <v-layout row align-center justify-center>
                      <v-flex xs6>
                          <span> No widget found! Please add a new widget.</span>
                      </v-flex>

                      <v-flex xs6>
                          <img class=""
                            height=""
                            src="~/assets/images/help/add_chart.gif"
                          >
                      </v-flex>
                    </v-layout>

                    </v-alert>

                </v-card>


                <div ref="gridLayout">

                    <grid-layout :layout="widgetLayout" :col-num="numberOfTotalColumns" :col-width="widgetNumberOfColums" :row-height="rowHeight" :margin="[widgetMarginX, widgetMarginY]" :is-draggable="true" :is-resizable="true" :is-mirrored="false" :vertical-compact="true"
                        :use-css-transforms="false" @layout-updated="layoutUpdatedEvent" class="">

                        <grid-item v-for="item in widgetLayout" :key="item.i" :x="item.x" :y="item.y" :w="item.w" :h="item.h" :i="item.i" :autoSize="true" @resize="resizeEvent" @resized="resizedEvent" @move="moveEvent" @moved="movedEvent" dragAllowFrom=".vue-draggable-handle">

                            <div style="border: 1px solid green">

                                <sciviz-chart v-if="item.type == 'chart'"
                                  :type="item.type"
                                  :id="item.widgetId"
                                  :title="item.i"
                                  :width="item.chartWidth"
                                  :height="item.chartHeight"
                                  :chartSpec="item.chartSpec"
                                  @onClickDelete="onChartDeleteButton"
                                  @onClickEdit="onChartEditButton"
                                  @onClickSave="onChartSaveButton"
                                  @onClickAddFilter="onChartAddFilterButton"
                                  />

                                <data-table v-if="item.type == 'table'" :type="item.type" :id="item.widgetId" :title="item.i" :width="item.chartWidth" :height="item.chartHeight" @onClickDelete="onChartDeleteButton" />

                            </div>

                        </grid-item>

                    </grid-layout>
                </div>

                <!-- speed dial menu -->
                <v-speed-dial v-model="speedDialMenu" direction="top" transition="slide-y-reverse-transition" :open-on-hover="false" fixed bottom right>
                    <v-btn slot="activator" v-model="speedDialMenu" color="blue darken-2" dark fab>
                        <v-icon>add</v-icon>
                        <v-icon>close</v-icon>
                    </v-btn>


                    <v-tooltip left>
                        <v-btn fab dark small color="green" @click="addWidget('chart')" slot="activator">
                            <v-icon>show_chart</v-icon>
                        </v-btn>
                        <span>Add a chart</span>
                    </v-tooltip>

                    <v-tooltip left>
                        <v-btn fab dark small color="green" @click="addWidget('table')" slot="activator">
                            <v-icon>table_chart</v-icon>
                        </v-btn>
                        <span>Add a table</span>
                    </v-tooltip>

                    <v-tooltip left>
                        <v-btn fab dark small color="green" @click="addDataset('csv')" slot="activator">
                            <v-icon>create_new_folder</v-icon>
                        </v-btn>
                        <span>Add a dataset</span>
                    </v-tooltip>


                </v-speed-dial>


            </v-flex>
        </v-layout><!-- gridLayout -->

    </div>
</template>

<script>



if (process.browser) {
  var VueGridLayout =  require('vue-grid-layout');
// import VueGridLayout from 'vue-grid-layout'
var GridLayout = VueGridLayout.GridLayout;
var GridItem = VueGridLayout.GridItem;
}



import ChartWidget from '~/components/charts/ChartWidget'
import DataTableWidget from '~/components/tables/DataTableWidget'
import store from '@/store'

export default {
    name: 'WidgetFactory',
    props: ['type'],
    components: {
        'grid-layout': GridLayout,
        'grid-item': GridItem,
        'sciviz-chart': ChartWidget,
        'data-table': DataTableWidget
    },
    data() {
        return {
            numberOfTotalColumns: 12, // number of columns in the grid
            widgetNumberOfColums: 4, // width of the widget in number of columns
            rowHeight: 40, // height of the widget
            widgetNumberOfRows: 10, // widget height = rowHeight x widgetNumberOfRows
            widgetMarginX: 0, // margin between the widgets in pixels
            widgetMarginY: 0,
            speedDialMenu: null
        }
    },
    beforeCreate() {
        console.log('called WidgetFactory::beforeCreate()')
        // console.log(this.widgetLayout)
    },
    created() {
        console.log('called WidgetFactory::created()')
        // console.log(this.widgetLayout)
        // this.widgetLayout[0].chartWidth = this.widgetLayout[0].w*this.widgetNumberOfColums
        // this.widgetLayout[0].chartWidth *= 2
    },
    mounted() {
        // touch support
        // https://github.com/jbaysolutions/vue-grid-layout/issues/110
    },
    computed: {

        widgetLayout (){
            return this.$store.getters.widgets
        },
        numberOfTotalWidgets (){
            return this.$store.getters.numberOfWidgets
        },
        numberOfTotalDatasets (){
            return this.$store.getters.numberOfDatasets
        }
    },
    methods: {
        layoutUpdatedEvent(newLayout) {
            console.log("Updated chart layout: ", newLayout)
        },
        moveEvent(i, newX, newY) {
            console.log("WIDGET MOVE i=" + i + ", X=" + newX + ", Y=" + newY);
        },
        resizeEvent(i, newH, newW, newHPx, newWPx) {
            console.log("WIDGET RESIZE i=" + i + ", H=" + newH + ", W=" + newW + ", H(px)=" + newHPx + ", W(px)=" + newWPx);
            this.widgetLayout[i].chartWidth = newWPx
            this.widgetLayout[i].chartHeight = newHPx
        },
        movedEvent(i, newX, newY) {
            console.log("WIDGET MOVED i=" + i + ", X=" + newX + ", Y=" + newY);
        },
        resizedEvent(i, newH, newW, newHPx, newWPx) {
            console.log("WIDGET RESIZED i=" + i + ", H=" + newH + ", W=" + newW + ", H(px)=" + newHPx + ", W(px)=" + newWPx);
            console.log('widgetLayout: \n', this.widgetLayout[i])
            this.widgetLayout[i].chartWidth = newWPx
            this.widgetLayout[i].chartHeight = newHPx
        },
        addWidget(widgetType) {
            console.log('called WidgetFactory::addWidget()')
            console.log("widgetType:", widgetType)
            console.log('Adding a new widget')

            var widgetId = this.generateUuid()
            console.log('Widget id:', widgetId)

            const gridWidth = this.$refs.gridLayout.clientWidth;
            console.log('grid width:', gridWidth)

            const numberOfWidgetX = this.numberOfTotalColumns / this.widgetNumberOfColums;
            const widgetWidth = (gridWidth - (numberOfWidgetX - 1) * this.widgetMarginX) / numberOfWidgetX;

            console.log('numberOfWidgetX:', numberOfWidgetX)
            console.log('widget width:', widgetWidth)

            var widgetTouch = {
                "type": widgetType,
                "x": 0,
                "y": 0,
                "w": this.widgetNumberOfColums,
                "h": this.widgetNumberOfRows,
                "i": this.widgetLayout.length,
                "chartWidth": widgetWidth,
                "chartHeight": 400,
                "widgetId": widgetId,
                "sm-x": 1,
                "sm-y": 1,
                "sm-w": 12,
                "sm-h": 4,
                "sm-draggable": false,
                "sm-resizable": false
            }

            if ( widgetType == 'chart' ) {
                widgetTouch = {...widgetTouch,
                     "chartType": "",
                    "chartSpec": this.$store.getters.vegaSpec
                  }
             }

            this.$store.commit('addChart', widgetTouch)
            // this.widgetLayout.push(widgetTouch)
            // console.log(this.widgetLayout)

        },
        addDataset(addDataset) {
          console.log('called WidgetFactory::addDataset()')
          console.log("dataType:", addDataset)
          console.log('Adding a new dataset.')

          console.log('called App:loadData()')
          const datasetId =  this.$store.getters.data.length + 1
          const testData = {
            id: datasetId,
            name: 'test data' + datasetId,
            url: 'www.google.com',
            enabled: false
          }
          console.log('Called App::loadData()')
          this.$store.commit('addDataset', testData)

          console.log('store:data', this.$store.getters.data[0])
        },
        onChartDeleteButton(signalValue, widgetId) {
            console.log('called onChartDeleteButton()')

            const index = this.widgetLayout.findIndex(widget => widget.widgetId === widgetId)
            this.widgetLayout.splice(index, 1)
        },
        onChartEditButton(signalValue, widgetId) {
            console.log('called onChartEditButton()')
        },
        onChartSaveButton(signalValue, widgetId) {
            console.log('called onChartSaveButton()')
        },
        onChartAddFilterButton(signalValue, widgetId) {
            console.log('called onChartAddFilterButton()')

            const index = this.widgetLayout.findIndex(widget => widget.widgetId === widgetId)
            // this.widgetLayout[index].chartSpec.mark = "circle"

            var newFilter = { "brushX": {"type": "interval", "encodings": ["x"]} }
            // var newFilter = {
            //   "brush": {
            //     "type": "interval",
            //     "on": "[mousedown[event.shiftKey], window:mouseup] > window:mousemove!",
            //     "translate": "[mousedown[event.shiftKey], window:mouseup] > window:mousemove!",
            //     "zoom": "wheel![event.shiftKey]"
            //   }
            // }

            //                 "resolve": "union",
           this.$store.commit('addChartFilter', newFilter)

            // this.widgetLayout[index].chartSpec = {...this.widgetLayout[index].chartSpec,
            //     "selection": {
            //       ...newFilter
            //     }
            // }


            this.widgetLayout[index].chartSpec = {...this.widgetLayout[index].chartSpec,
                "selection": {
                  ...this.$store.getters.vegaFilters[0]
                }
            }





            if ( this.$store.getters.numberOfWidgets > 1 ) {

              console.log('Found', this.$store.getters.numberOfVegaFilters, 'filters.')
              console.log('Applying the filter:')
              console.log(this.$store.getters.vegaFilters[0])

              // this.widgetLayout[index].chartSpec = {...this.widgetLayout[index].chartSpec,
              //     "color": {
              //       "condition": {"selection": this.$store.getters.vegaFilters[0], "field": "Origin", "type": "nominal"},
              //       "value": "grey"
              //     }
              // }

              // var newSelector = {
              //     "scale": {
              //       "domain":
              //         {
              //           "selection": this.$store.getters.widgets[0].chartSpec.selection[0]
              //         }
              //     }
              // }


              var newSelector = {
                  "scale": {
                    "domain":
                      {
                        "selection": this.$store.getters.vegaFilters[0]
                      }
                  }
              }


              // this.widgetLayout[index].chartSpec.encoding.x = {...this.widgetLayout[index].chartSpec.encoding.x,
              //     ...newSelector
              // }

              this.$store.commit('applyChartFilter', {widgetId, newSelector})

            }





        },
        generateUuid() {
            console.log('called generateUuid()')
            return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
                var r = Math.random() * 16 | 0,
                    v = c == 'x' ? r : (r & 0x3 | 0x8);
                return v.toString(16);
            })
        }
    }
}
</script>

<style>
.vue-grid-item {
  @media screen and (max-width: $mobile-break) {
    width: 100%;
    height: auto;
    position: relative;
    margin-top: $padding;
    transform: inherit;
  }
}

.vue-grid-item.vue-grid-placeholder {
  background: white;
}

.vue-grid-item.vue-grid-placeholder {
  background-color: white;
}
</style>
