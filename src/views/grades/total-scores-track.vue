<template>
	<div>
		<div class="ui-search-wrap" id="ui-search-wrap">
			<el-form :inline="true">

				<el-form-item label="届">
          <el-select v-model="listQuery.period" filterable clearable placeholder="请选择" @change="getList('period')">
            <el-option v-for="item in periodList" :label="item.label" :value="item.label" :key="item.value">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="年级">
          <el-select v-model="listQuery.grade" filterable clearable placeholder="请选择" @change="getList('grade')">
            <el-option v-for="item in gradeList" :label="item.label" :value="item.label" :key="item.label">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </div>
		<div class="ui-table-wrap clearfix">
			<h3 class="ui-table-title">
				<wscn-icon-svg icon-class="shuxian"/>
				{{name}}
			</h3>
			<div class="ui-table-main">
				<el-table v-if="!listLoading" v-loading.body="listLoading" :data="list.data" border style="width: 100%" >
	        <el-table-column v-for='(first,index) in list.head' :label="first.name" :key='first.name' sortable>
	          <el-table-column v-if="first.children != undefined" v-for='(second,index) in first.children' :label="second.name" :key='second.name' sortable>
		            <template scope="scope">
		              <div>{{scope.row[first.value][second.value]}}</div>
		            </template>
		          </el-table-column>

		          <template scope="scope" v-if="first.children == undefined">
		            <div>{{scope.row[first.value]}}</div>
		          </template>
		        
		        </el-table-column>
		    </el-table>
			</div>
			<div v-show="!listLoading" class="page-wrap fr">
	      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.pageNo" :page-sizes="[30, 40, 50, 60, 70, 80]"
	        :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
	      </el-pagination>
	    </div>
	  </div>
	</div>
</template>
<script>
  import { getLatestTest, attrPeriod, attrGrade } from 'utils/auth';
	import { mapGetters } from 'vuex';
	import { gradeList, periodList } from 'utils/data';
	import { quaryAllScoreRataByPeriodForPage } from 'api/excellent';
	export default {
		data() {
			return {
				name: '总分跟踪管理',
				periodList: periodList(),
        gradeList: [],
				screenHeight: 0,
				list: {
					data: [],
					head: []
				},
				total: null,
        listLoading: true,
        listQuery: {
          period: '',
          pageNo: 1,
          pageSize: 50,
          grade: '九年级',
          subject: '',
        }
			}
		},
		computed: {
      ...mapGetters([
        'subject',
        'gradeNo'
      ])
    },
		created() {

    },
		mounted() {
			this.setForm();
			this.setDefault();
		},
		methods: {
			setForm(){
				let grade_list = gradeList('all');
	      for(let i=0; i<grade_list.length; i++){
	        for(var o=0; o<grade_list[i].options.length; o++){
	          this.gradeList.push(grade_list[i].options[o]);
	        }
	      };
			},
			setDefault(){
				
				this.listQuery.subject = this.subject;
				if(typeof attrGrade() != 'undefined') this.listQuery.grade = attrGrade();
				if(typeof attrPeriod() != 'undefined') this.listQuery.period = attrPeriod();
				else this.listQuery.period = this.periodList[this.periodList.length-1].value;

				this.getList();

			},
			getList(type) {
				switch(type){
          case 'period':
            attrPeriod(this.listQuery.period);
            break;
          case 'grade':
            attrGrade(this.listQuery.grade);
            break;
        }
        this.listLoading = true;
        quaryAllScoreRataByPeriodForPage(this.listQuery).then(res => {
        	if(typeof res == 'undefined'){
        		this.listLoading = false;
        		this.list.data = [];
        		return
        	}
          var data = res.data.data;
          this.list.data = data.data;
          this.list.head = data.head;
          this.total = data.total;
          this.listLoading = false;
        })
      },
      handleSizeChange(val) {
        this.listQuery.pageSize = val;
        this.getList();
      },
      handleCurrentChange(val) {
        this.listQuery.pageNo = val;
        this.getList();
      }
		}
	}
</script>