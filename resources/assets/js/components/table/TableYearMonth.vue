<style scoped>

  .button-size{
    width: 50px;
  }

  tr{
    cursor: pointer;
  }

  .table-hscroll{
     white-space:nowrap;
     overflow-x:auto; 
     overflow-y:auto; 
     width:auto; 
  }

  .table-height{
    height:auto;
  }

  .selectwidth{
    position: relative;
    min-width:80px;
    margin-right: 5px;
  }

</style>

<template>

<div class="container-fluid">
  <div class="row">
   
      <div class="panel panel-default" > 
        <div class="panel-heading">
          <h3 class="panel-title">
            {{ ts[tableTitle] }}
            <span style="color:blue; padding-top:10px; padding-right:20px" align="left" v-if='loading'>
              <img src="/assets/icons/loading_image.gif"/>   
            </span>
          </h3>
        </div> 
        <div class="panel-body">
          <div class="row">
            <div v-if="showYear">
              <div class="col-xs-1 selectwidth " v-if="showYear">
                {{ ts['year'] }}: 
                <select name="year" class="form-control selectwidth" v-model="yearSelected" @change="changeYearMonth()">
                  <option v-for="(key, value) in years" value="{{value}}"> {{value}} </option>
                </select>
              </div>
              <div class="col-xs-1" v-if="showMonth" >
                {{ ts['month'] }}:  
                <select name="month" class="form-control selectwidth" v-model="monthSelected" @change="changeYearMonth()">
                  <option v-for="(key, value) in months" value="{{key + 1}}"> {{value}} </option>
                </select>
              </div>
            </div>
          </div>
          <br>
          <div  class="table-responsive table-hscroll table-height">
            <table id="{{id}}" class= "table table-striped table-bordered table-condensed table-hover" >
              <col v-for="col in cols" :width="col.width">
              <!--/div-->
              <thead>
                <tr>
                  <th class="header" v-for="col in cols"> 
                       {{ ts[col.name] }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="1 in 10">
                  <td v-for="1 in cols.length"> &nbsp; </td>
                </tr> 
                  
                <tr v-for="row in rows"> 
                  <td  v-for="(key, value) in row" @click="itemSelected(row)">
                    <span v-if="key=='deleted_at'" >
                        <span v-if="value==undefined">
                            <span class="btn btn-xs btn-success"> {{ ts['active'] }}  </span>
                           <!--  <i class="glyphicon glyphicon-ok"></i> -->
                        </span>
                        <span v-else>
                            <span class="btn btn-xs btn-danger"> {{ ts['inactive'] }} </span>
                            <!-- <i class="glyphicon glyphicon-remove" ></i> -->
                        </span>
                    </span>

                    <span v-else>
                        {{value}} 
                    </span>

                  </td>
              
                  <td v-if="(arrIconInfo.length > 0)">
                    <span v-for="info in arrIconInfo">
                      <a href="{{info.url}}" title="{{info.title}}" @click.prevent="clickIcon(row, info)"> <i class="{{info.icon}}"></i> {{info.text}} </a>
                    </span>
                  </td>

                  <td v-if="(arrIconActions.length > 0)">
                    <span v-for="action in arrIconActions">
                      <a href="{{action.url}}" title="{{action.title}}" @click.prevent="clickIcon(row, action)"> <i class="{{action.icon}}"></i> {{action.text}} </a>
                    </span>
                  </td>
                </tr> 
               </tbody> 
            </table> 
          </div>
          <div > 

          </div>
            <div class="row">
               <div class="col-sm-6">
                <div align="left" style="padding-top:10px">
                  <p style="font-size:13px;" > 
                     {{ ts['showing'] }}: {{from}} {{ ts['to'] }} {{to}} {{ ts['of'] }} {{total}} items
                  </p>
                </div>
              </div>
              <div v-if="NoMorePages" style="color:gray" align="right"> 
                  <strong> <em>{{ ts['noMorePages'] }}</strong></em> 
              </div>  
              <div class="col-sm-6">
                <div align="right" style="padding-top:10px">
                  <a class="btn btn-sm btn-primary button-size" @click.prevent="goToFisrtPage"> {{ ts['start'] }} </a> 
                  <a class="btn btn-sm btn-primary button-size" @click.prevent="goToPrevPage"> {{ ts['prev'] }} </a> 
                  <a class="btn btn-sm btn-primary button-size" @click.prevent="goToNextPage"> {{ ts['next'] }} </a> 
                  <a class="btn btn-sm btn-primary button-size" @click.prevent="goToLastPage"> {{ ts['end'] }} </a> 
                  <br/> <br/>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- <pre>{{ $data | json }}</pre>  -->
    </div>
</div>

</template>


<script>

 import MyLang from '../../components/languages/Languages.vue';
  
  module.exports = {

    mixins: [MyLang],

    props: ['tableId', 'tableTitle', 'columnsNames', 'url', 'iconInfo', 'iconActions', 'showYear', 'showMonth'],

    ready: function(){
      this.id=this.tableId;
      this.cols=this.jsonToArray(this.columnsNames);
      this.arrIconInfo=this.jsonToArray(this.iconInfo);
      this.arrIconActions=this.jsonToArray(this.iconActions);
      this.showYear = (this.showYear=="false") ? false : true;
      this.showMonth =(this.showMonth=="false") ? false : true;
      this.initialUrl = this.url;
      this.lastUrl =this.initialUrl;
      this.initYearandMonth();
      this.readPageData(this.lastUrl, this.yearSelected, this.monthSelected, false);
    },

    data: function() {
      return {
        id:'',
        initialUrl: '',
        lastUrl: '',
        current_page:'',
        per_page:'',
        first_page: 1,
        last_page:'',
        to:'',
        from:'',
        first_page_url:'',
        next_page_url:'',
        prev_page_url:'',
        last_page_url:'',
        total:'',
        NoMorePages:false,
        loading: false,
        cols: [],
        rows:[],
        arrIconInfo:[],
        arrIconActions:[],
        filterApplied: '',
        years:[],
        months: [],
        yearSelected:'', 
        monthSelected:'',
      }
    },

    methods: {

      clickIcon: function(row, link){
        this.$route.router.go(link.url);
      },

      goToFisrtPage: function(){
        this.goToNextPrev(this.first_page_url, this.first_page);
      },

      goToPrevPage: function(){
        this.goToNextPrev(this.prev_page_url, this.first_page);
      },

      goToNextPage: function(){
        this.goToNextPrev(this.next_page_url, this.last_page);
      },

      goToLastPage: function(){
        this.goToNextPrev(this.last_page_url, this.last_page);
      },

      goToPageNumber: function($page){
        this.readPageData(this.lastUrl + '?page=' + $page, false);
        this.NoMorePages=false;
      },

      goToNextPrev: function(goToUrl , pageNumber){

        if (pageNumber!=this.current_page){
          if (goToUrl!=="null") {
            this.readPageData(goToUrl, this.yearSelected, this.monthSelected, false);
            this.NoMorePages=false;
          }else
            this.NoMorePages=true;
        }else{
          this.NoMorePages=true;
        }
      },

      readPageData: function(url, year, month, displayMsg){
        var self=this;
        self.NoMorePages=false;
        self.loading= true;
        self.$http.get(url, { params: {"year": year, "month": month} } ).then(function (response){
          self.setDataResponse(response.data);
          self.routesFirstPrevNextLast(response.data);
          $('#' + self.id + ' td').remove(); 
        }).then(function (response) {
          self.loading= false;
        }).catch(function (response) {
          self.displayPopUpMessage(response);
          self.loading= false;
        });
      },

       setDataResponse: function (response){
        this.rows = response.data;
        this.total = response.total;
        this.per_page = response.per_page;
        this.current_page = response.current_page;
        this.last_page = response.last_page;
        if(response.to>0)
          this.from = response.from;
        else
          this.from = 0;
        this.to = response.to;
      },

      routesFirstPrevNextLast: function(response){
        //get url without parameters
        if (response.next_page_url){
          var page_url=response.next_page_url; 
        }
        else{
          var page_url=response.prev_page_url; 
        }
        //set the route for first, prev, next, last page action
        if (page_url){

          this.first_page_url=page_url.slice(0, page_url.search('page')) + 'page=' + this.first_page + this.setSearchParam();
          this.next_page_url=response.next_page_url + this.setSearchParam();
          this.prev_page_url=response.prev_page_url + this.setSearchParam();
          this.last_page_url=page_url.slice(0, page_url.search('page')) +  'page=' + this.last_page + this.setSearchParam();
         }
      },

      setSearchParam: function(){
        var searchParam='';

        if (this.showYear){
          searchParam= '&&year=' + this.yearSelected;
          if (this.showMonth)
            searchParam= searchParam +'&&month=' + this.monthSelected;

        } 

        return searchParam;
      },
      
      initYearandMonth: function(){
        var date = new Date();
        var currentYear = date.getFullYear();

        for (var i = currentYear; i >= currentYear - 5; i--) { 
          this.years.push(i);
        }

        this.months = this.ts['short_months'];
        
        this.yearSelected = currentYear;
        this.monthSelected = date.getMonth() + 1;
      },

      changeYearMonth: function(){
        this.readPageData(this.lastUrl, this.yearSelected, this.monthSelected, false);
      },

      displayPopUpMessage: function(response){
        try{
          this.$dispatc('displayAlert', (!response.data.error) ? 'success' : 'danger', response.data.message + ' (' + response.status + ')');
        }catch(error){
          this.$dispatch('displayAlert', 'danger', error.name + ' Exception: ' + error.message);
        }
      },

      jsonToArray: function(data){
        var parsed = JSON.parse(data);
        var arr = [];
        for(var x in parsed){
          arr.push(parsed[x]);
        }

        return arr;
      }
    },
    
  }
</script>