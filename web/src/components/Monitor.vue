<template>
  <el-container style="padding: 0px;">
    <el-header style="padding-top: 24px; padding-left: 24px;">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item @click.native="handleBreadcrumb()" style="cursor: pointer;">NATS-WebUI</el-breadcrumb-item>
        <el-breadcrumb-item @click.native="handleBreadcrumb()" style="cursor: pointer">Servers</el-breadcrumb-item>
        <el-breadcrumb-item>{{server.name}}</el-breadcrumb-item>
      </el-breadcrumb>
    </el-header>
    <el-main style="display: flex; flex-direction: row; padding: 0px; flex: 1 1 100%;">
      <div id="details" style="flex: 1 1 100%; text-align: left; padding: 0px 24px;">
        <div style="display: flex; align-items: flex-end; margin-bottom: 12px;">
          <h1 style="font-size: 3em; margin: 0px;">
            {{server.name}}
          </h1>
          <span style="margin-bottom: 8px; margin-left: 8px;">
            @ {{ server.host }}{{ server.port === 4222 ? '' : (':' + server.port) }}
          </span>
        </div>
        
        <div v-if="server.varz!==null" style="display: flex; flex-direction: column;">
          <div class="metric-row">
            <div class="metric">
              <h1 class="metric-label">Total Messages In</h1>
              <span class="metric-value">{{ server.varz.in_msgs | numeral('0.[00]a') }}</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Total Messages Out</h1>
              <span class="metric-value">{{ server.varz.out_msgs | numeral('0.[00]a') }}</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Total Data Volume In</h1>
              <span class="metric-value">{{ server.varz.in_bytes | numeral('0.00b') }}</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Total Data Volume Out</h1>
              <span class="metric-value">{{ server.varz.out_bytes | numeral('0.00b') }}</span>
            </div>
          </div>
          <div class="metric-row">
            <div class="metric">
              <h1 class="metric-label">Message In Rate</h1>
              <span class="metric-value">{{ in_msgs_rate | numeral('0.[00]a') }}</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Message Out Rate</h1>
              <span class="metric-value">{{ out_msgs_rate | numeral('0.[00]a') }}</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Incoming Data Rate</h1>
              <span class="metric-value">{{ in_bytes_rate | numeral('0.00b') }}/s</span>
            </div>
            <div class="metric">
              <h1 class="metric-label">Outgoing Data Rate</h1>
              <span class="metric-value">{{ out_bytes_rate | numeral('0.00b') }}/s</span>
            </div>
          </div>
          <div style="display: flex; flex-direction: row; width: 100%;">
            <VueApexCharts style="flex: 1 1 50%;" v-show="msgs_in_series.length > 1" width="100%" height="180" type="area" ref="chart1" :options="chart1Options" :series="series1"></VueApexCharts>
            <VueApexCharts style="flex: 1 1 50%;" v-show="bytes_in_series.length > 1" width="100%" height="180" type="area" ref="chart2" :options="chart2Options" :series="series2"></VueApexCharts>
          </div>
          <div class="metric-row">
            <div class="metric-smaller">
              <h1 class="metric-label">CPU Usage</h1>
              <span class="metric-value-smaller">{{ server.varz.cpu | numeral('0.[00]a') }} %</span>
            </div>
            <div class="metric-smaller">
              <h1 class="metric-label">Mem Usage</h1>
              <span class="metric-value-smaller">{{ server.varz.mem | numeral('0.00b') }}</span>
            </div>
            <div class="metric-smaller">
              <h1 class="metric-label">Connections</h1>
              <span class="metric-value-smaller">{{ server.varz.connections }}</span>
            </div>
            <div class="metric-smaller">
              <h1 class="metric-label">Subscriptions</h1>
              <span class="metric-value-smaller">{{ server.varz.subscriptions }}</span>
            </div>
            <div class="metric-smaller">
              <h1 class="metric-label">Slow Consumers</h1>
              <span class="metric-value-smaller">{{ server.varz.slow_consumers }}</span>
            </div>
            <div class="metric-smaller">
              <h1 class="metric-label">Uptime</h1>
              <span class="metric-value-smaller">{{ server.varz.uptime }}</span>
            </div>
          </div>
          <div id="small-metrics" style="display: flex; flex-direction: row; flex-wrap: wrap;">
            <div class="small-metric">
              <span class="metric-label-small">
                NATS Version
              </span>
              <span class="metric-value-small">
                {{ server.varz.version }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Protocol
              </span>
              <span class="metric-value-small">
                {{ server.varz.proto }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Go Version
              </span>
              <span class="metric-value-small">
                {{ server.varz.go }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Maximum Connections
              </span>
              <span class="metric-value-small">
                {{ server.varz.max_connections }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Ping Interval
              </span>
              <span class="metric-value-small">
                {{ server.varz.ping_interval / 1000000000 }}s
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Ping Max
              </span>
              <span class="metric-value-small">
                {{ server.varz.ping_max }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Max Control Line
              </span>
              <span class="metric-value-small">
                {{ server.varz.max_control_line }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Maximum Payload Size
              </span>
              <span class="metric-value-small">
                {{ server.varz.max_payload | numeral('0.00b') }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Maximum Pending
              </span>
              <span class="metric-value-small">
                {{ server.varz.max_pending | numeral('0.00b') }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                TLS Timeout
              </span>
              <span class="metric-value-small">
                {{ server.varz.tls_timeout }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Write Deadline
              </span>
              <span class="metric-value-small">
                {{ server.varz.write_deadline / 1000000000 }}s
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Start Time
              </span>
              <span class="metric-value-small">
                {{ server.varz.start | moment("MMMM Do YYYY, h:mm:ss a") }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                CPU Cores
              </span>
              <span class="metric-value-small">
                {{ server.varz.cores }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Total Connections
              </span>
              <span class="metric-value-small">
                {{ server.varz.total_connections }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Routes
              </span>
              <span class="metric-value-small">
                {{ server.varz.routes }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Remotes
              </span>
              <span class="metric-value-small">
                {{ server.varz.remotes }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Leaf Nodes
              </span>
              <span class="metric-value-small">
                {{ server.varz.leafnodes }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                NATS Port
              </span>
              <span class="metric-value-small">
                {{ server.port }}
              </span>
            </div>
            <div class="small-metric">
              <span class="metric-label-small">
                Monitoring Port
              </span>
              <span class="metric-value-small">
                {{ server.monitoring_port }}
              </span>
            </div>
            <!-- <div class="small-metric">
              <span class="metric-label-small">
                Server Name
              </span>
              <span class="metric-value-small">
                {{ server.varz.server_name }}
              </span>
            </div> -->
            <div class="small-metric">
              <span class="metric-label-small">
                Server ID
              </span>
              <span class="metric-value-small">
                {{ server.varz.server_id}}
              </span>
            </div>
          </div>
        </div>
        <div style="text-align: center; color: #909399; font-size: 14px;" v-else>The NATS Server could not be reached.</div>
      </div>
      <div id="subjects" style="flex: 1 0 360px; padding: 0px; text-align: left; overflow-y: auto;">
        <div style="display: flex; flex-direction: row; justify-content: space-between; align-items: center; padding-right: 24px;">
          <h1 style="margin: 8px 0px;">Server Subject Hierarchy</h1>
          <el-button @click="editSubjectHierarchy" icon="el-icon-edit">Edit</el-button>
        </div>
        <el-tree :data="server.subjects" empty-text="No subjects configured for this server." default-expand-all
          :props="{label: 'subject_str', children: 'subjects', disabled: false, isLeaf: checkIsLeaf}">
        </el-tree>
        <!-- <h1>Publications</h1>
        <el-tree
          :data="data"
          show-checkbox
          node-key="id"
          :default-expanded-keys="[2, 3]"
          :default-checked-keys="[5]"
          :props="defaultProps">
        </el-tree> -->
      </div>
    </el-main>
    <el-dialog title="Edit Subject Tree" :visible.sync="subjectHierarchyDialogVisible" width="30%" style="height: auto;" center>
      <div style="overflow-y: auto; overflow-x: auto; max-height: 50vh;">
        <AceEditor width="100%" height="40vh" :fontSize="14" :showPrintMargin="true" :showGutter="true" :value="tabtree" mode="plain_text" theme="chrome" :onChange="handleSubjectHierarchyChange"></AceEditor>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="handleSubjectHierarchyDialogCancel">Cancel</el-button>
        <el-button type="primary" @click="saveSubjectTree">Save</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
import { mapState, mapActions, mapMutations } from 'vuex'
import tabdown from 'tabdown-sacha'
import {Ace as AceEditor} from 'vue2-brace-editor'
import VueApexCharts from 'vue-apexcharts'

import 'brace/mode/plain_text'
import 'brace/theme/chrome'

export default {
  components: {
    AceEditor,
    VueApexCharts
  },
  computed: {
    ...mapState({
      server: s => s.transient.serversMap[s.transient.selectedServer],
      selected_index: s => s.transient.selectedServer
    })
  },
  data() {
    return {
      tabtree: "",
      subjectHierarchyDialogVisible: false,
      msgs_in_series: [],
      msgs_out_series: [],
      bytes_in_series: [],
      bytes_out_series: [],
      in_msgs_rate: 0,
      out_msgs_rate: 0,
      in_bytes_rate: 0,
      out_bytes_rate: 0,
      chart1Options: {
        chart: {
          id: 'realtime',
          height: 180,
          type: 'line',
          animations: {
            enabled: true,
            easing: 'linear',
            dynamicAnimation: {
              speed: 1000
            }
          },
          toolbar: {
            show: false
          },
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'smooth'
        },
        title: {
          text: 'Message Rate',
          align: 'left'
        },
        markers: {
          size: 0
        },
        xaxis: {
          type: 'datetime',
          range: 60000,
        },
        yaxis: {
          min: 0,
          tickAmount: 3,
          forceNiceScale: true
        },
        legend: {
          show: false
        },
      },
      series1: [
        { 
          name: 'Messages In',
          data: []
        },
        {
          name: 'Messages Out',
          data: []
        }
      ],
      chart2Options: {
        chart: {
          id: 'realtime',
          height: 180,
          type: 'line',
          animations: {
            enabled: true,
            easing: 'linear',
            dynamicAnimation: {
              speed: 1000
            }
          },
          toolbar: {
            show: false
          },
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'smooth'
        },
        title: {
          text: 'Traffic Volume',
          align: 'left'
        },
        markers: {
          size: 0
        },
        xaxis: {
          type: 'datetime',
          range: 60000,
        },
        yaxis: {
          min: 0,
          tickAmount: 3,
          forceNiceScale: true
        },
        legend: {
          show: false
        },
      },
      series2: [
        { 
          name: 'Bytes In',
          data: []
        },
        {
          name: 'Bytes Out',
          data: []
        }
      ],
    }
  },
  watch: {
    "server.varz": function (v, u) {
      if (u !== undefined && !this.subjectHierarchyDialogVisible) {
        let dt = new Date().getTime();
        this.in_msgs_rate = v.in_msgs - u.in_msgs
        this.msgs_in_series.push({
          x: dt,
          y: this.in_msgs_rate
        })
        this.out_msgs_rate = v.out_msgs - u.out_msgs
        this.msgs_out_series.push({
          x: dt,
          y: this.out_msgs_rate
        })
        this.in_bytes_rate = v.in_bytes - u.in_bytes
        this.bytes_in_series.push({
          x: dt,
          y: this.in_bytes_rate
        })
        this.out_bytes_rate = v.out_bytes - u.out_bytes
        this.bytes_out_series.push({
          x: dt,
          y: this.out_bytes_rate
        })

        let animflag = true

        if (this.msgs_in_series.length > 180) {
          this.msgs_in_series = this.msgs_in_series.slice(this.msgs_in_series.length - 61)
          animflag = false
        }
        if (this.msgs_out_series.length > 180) {
          this.msgs_out_series = this.msgs_out_series.slice(this.msgs_out_series.length - 61)
          animflag = false
        }
        if (this.bytes_in_series.length > 180) {
          this.bytes_in_series = this.bytes_in_series.slice(this.bytes_in_series.length - 61)
          animflag = false
        }
        if (this.msgs_in_series.length > 180) {
          this.bytes_out_series = this.bytes_out_series.slice(this.bytes_out_series.length - 61)
          animflag = false
        }
        
        this.$refs.chart1.updateSeries([
          { 
            name: 'Messages In',
            data: animflag ? this.msgs_in_series : []
          },
          {
            name: 'Messages Out',
            data: animflag ? this.msgs_out_series : []
          }
        ], animflag)
        this.$refs.chart2.updateSeries([
          { 
            name: 'Bytes In',
            data: animflag ? this.bytes_in_series : []
          },
          {
            name: 'Bytes Out',
            data: animflag ? this.bytes_out_series : []
          }
        ], animflag)
      }
    },
    "selected_index": function () {
      this.clearTimeSeriesHistory()
    },
    "subjectHierarchyDialogVisible": function () {
      this.clearTimeSeriesHistory()
    }
  },
  methods: {
    ...mapActions(['updateServer', 'getAppState']),
    ...mapMutations(['selectScreen']),
    editSubjectHierarchy() {
      this.tabtree = ""
      tabdown.traverse(this.subjectTreeToTabdownTreeRoot(this.server.subjects), function (node) {
        this.tabtree = this.tabtree.concat(this.tabtree.length === 0 ? '' : '\n','\t'.repeat(node.depth) + node.data)
      }.bind(this))
      // this.tabtree = tabdown.print(this.subjectTreeToTabdownTreeRoot(this.server.subjects))
      this.subjectHierarchyDialogVisible = true
    },
    async saveSubjectTree() {
      let server = JSON.parse(JSON.stringify(this.server))
      server.subjects = this.tabdownTreeToSubjectTree(tabdown.parse(this.tabtree.replace(/ {2}/g, '\t').split('\n'), '\t'), "")
      await this.updateServer(server)
      await this.getAppState()
      this.subjectHierarchyDialogVisible = false
    },
    subjectTreeToTabdownTree(tree, parent, depth) {
      let node = {
        data: tree.subject_str,
        depth: depth,
        parent: parent,
        children: [],
        tabcount: 0
      }
      for (var i in tree.subjects) {
        let child_node = tree.subjects[i]
        node.children.push(this.subjectTreeToTabdownTree(child_node, node, depth+1))
      }
      return node
    },
    subjectTreeToTabdownTreeRoot(nodes) {
      let node = {
        data: null,
        depth: -1,
        parent: null,
        children:[]
      }
      for (var i in nodes) {
        let child_node = nodes[i]
        node.children.push(this.subjectTreeToTabdownTree(child_node, node, 0))
      }
      return node
    },
    tabdownTreeToSubjectTree(tree, idPrefix) {
      let subjectTreeNodes = []
      for (var i in tree.children) {
        let node = tree.children[i]
        let subjectTreeNode = {
          id: idPrefix + i.toString(),
          subject_str: node.data,
          subjects: this.tabdownTreeToSubjectTree(node, idPrefix + i.toString() + "-"),
          selected: false
        }
        subjectTreeNodes.push(subjectTreeNode)
      }
      return subjectTreeNodes
    },
    checkIsLeaf() {
      return false
    },
    handleBreadcrumb() {
      this.selectScreen({isServer: true, index: -1})
    },
    handleSubjectHierarchyDialogCancel() {
      this.subjectHierarchyDialogVisible = false
    },
    clearTimeSeriesHistory() {
      this.msgs_in_series = []
      this.msgs_out_series = []
      this.bytes_in_series = []
      this.bytes_out_series = []
    },
    handleSubjectHierarchyChange(value) {
      this.tabtree = value
    }
  }
}
</script>

<style scoped>
h1.metric-label {
  margin: 0px;
  color: #C0C4CC;
}

span.metric-value {
  font-size: 2.5em;
  font-weight: 100;
}

span.metric-value-smaller {
  font-size: 2em;
  font-weight: 100;
}

.metric-row {
  display: flex; flex-direction: row nowrap;
  justify-content: space-between;
  margin-bottom: 24px;
}

.metric {
  display: flex;
  flex-direction: column;
  flex: 0 0 25%;
}

.metric-smaller {
  display: flex;
  flex-direction: column;
  flex: 0 0 16%;
}

#small-metrics {
  border: solid 1px rgb(230, 230, 230)
}

.small-metric:nth-child(even) {
  background: rgb(246, 246, 246);
}

.small-metric:nth-child(odd) {
  background: rgb(255, 255, 255);
}

.small-metric {
  display: flex;
  flex-direction: column;
  flex: 1 1 auto;
  padding-right: 48px;
  padding-left: 8px;
  padding-top: 12px;
  padding-bottom: 0px;
  border: solid 1px rgb(230, 230, 230);
}

.metric-label-small {
  font-size: x-small;
  white-space: nowrap;
}

.metric-value-small {
  font-size: normal;
  white-space: nowrap;
}
</style>