<template>
  <div>
    <el-table :data="proxies" :default-sort="{prop: 'name', order: 'ascending'}" style="width: 100%">
      <el-table-column type="expand">
        <template scope="props">
          <el-popover
            ref="popover4"
            placement="right"
            width="600"
  		  style="margin-left:0px"
            trigger="click">
            <my-traffic-chart :proxy_name="props.row.name"></my-traffic-chart>
          </el-popover>
  
          <el-button v-popover:popover4 type="primary" size="small" icon="view" style="margin-bottom:10px">Traffic Statistics</el-button>
  
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="Name">
              <span>{{ props.row.name }}</span>
            </el-form-item>
            <el-form-item label="Type">
              <span>{{ props.row.type }}</span>
            </el-form-item>
            <el-form-item label="Domains">
              <span>{{ props.row.custom_domains }}</span>
            </el-form-item>
            <el-form-item label="SubDomain">
              <span>{{ props.row.subdomain }}</span>
            </el-form-item>
            <el-form-item label="locations">
              <span>{{ props.row.locations }}</span>
            </el-form-item>
            <el-form-item label="HostRewrite">
              <span>{{ props.row.host_header_rewrite }}</span>
            </el-form-item>
            <el-form-item label="Encryption">
              <span>{{ props.row.encryption }}</span>
            </el-form-item>
            <el-form-item label="Compression">
              <span>{{ props.row.compression }}</span>
            </el-form-item>
            <el-form-item label="Last Start">
              <span>{{ props.row.last_start_time }}</span>
            </el-form-item>
            <el-form-item label="Last Close">
              <span>{{ props.row.last_close_time }}</span>
            </el-form-item>
        </el-form>
    </template>
    </el-table-column>
    <el-table-column
      label="Name"
      prop="name"
      sortable>
    </el-table-column>
    <el-table-column
      label="Port"
      prop="port"
      sortable>
    </el-table-column>
    <el-table-column
      label="Connections"
      prop="conns"
      sortable>
    </el-table-column>
    <el-table-column
      label="Traffic In"
      prop="traffic_in"
      :formatter="formatTrafficIn"
      sortable>
    </el-table-column>
    <el-table-column
      label="Traffic Out"
      prop="traffic_out"
      :formatter="formatTrafficOut"
      sortable>
    </el-table-column>
    <el-table-column
      label="status"
      prop="status"
      sortable>
      <template scope="scope">
        <el-tag type="success" v-if="scope.row.status === 'online'">{{ scope.row.status }}</el-tag>
        <el-tag type="danger" v-else>{{ scope.row.status }}</el-tag>
      </template>
    </el-table-column>
</el-table>
<pagination :totalPage="parentTotalPage" :currentPage="parentCurrentpage" :changeCallback="fetchData"></pagination> 
</div>
</template>

<script>
  import Humanize from 'humanize-plus';
  import pagination from '../utils/pagination.vue';
  import Traffic from './Traffic.vue';
  
  import {
    HttpProxy
  } from '../utils/proxy.js'
  export default {
    data() {
      return {
        proxies: null,
        vhost_https_port: "",
        subdomain_host: "",
        parentTotalPage: 1,
        parentCurrentpage: 1
      }
    },
    created() {
      this.fetchData(0)
    },
    watch: {
      '$route': 'fetchData'
    },
    methods: {
      formatTrafficIn(row, column) {
        return Humanize.fileSize(row.traffic_in)
      },
      formatTrafficOut(row, column) {
        return Humanize.fileSize(row.traffic_out)
      },
      fetchData(cPage) {
        fetch('/api/serverinfo', {credentials: 'include'})
          .then(res => {
            return res.json()
          }).then(json => {
            this.vhost_https_port = json.vhost_https_port
            this.subdomain_host = json.subdomain_host
            if (this.vhost_https_port == null || this.vhost_https_port == 0) {
              return
            } else {
              if (cPage == 0) {
                fetch('/api/proxy/https', {credentials: 'include'})
                  .then(res => {
                    return res.json()
                  }).then(json => {
                    this.parentTotalPage = json.total_page;
                    this.parentCurrentPage = 1;
                    
                    fetch('/api/proxy/https/1', {credentials: 'include'})
                    .then(res => {
                      return res.json()
                     }).then(json => {
                      this.proxies = new Array()
                      for (let proxyStats of json.proxies) {
                        this.proxies.push(new HttpsProxy(proxyStats, this.vhost_https_port, this.subdomain_host))
                      }
                    })
                  })
                } else {
                  this.parentCurrentpage = cPage;
                  fetch('/api/proxy/https/' + cPage, {credentials: 'include'})
                  .then(res => {
                    return res.json()
                  }).then(json => {                  
                    this.proxies = new Array()
                    for (let proxyStats of json.proxies) {
                      this.proxies.push(new HttpsProxy(proxyStats, this.vhost_https_port, this.subdomain_host))
                    }
                  })
                }
            }
          })
      }
    },
    components: {
        'my-traffic-chart': Traffic,
        'pagination': pagination
    }
  }
</script>

<style>
</style>
