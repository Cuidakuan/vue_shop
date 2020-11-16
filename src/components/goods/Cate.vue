<template>
    <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            </el-col>
        </el-row>

        <!-- 表格区域 -->
        <tree-table
        class="treeTable"
        :data='catelist'
        :columns='columns'
        :selection-type='false'
        :expand-type='false'
        :show-index='true'
        index-text='#'
        border>
        <!-- 是否有效列 -->
          <template slot="isok" slot-scope="scope">
              <i style="color:lightgreen;" class="el-icon-success" v-if="scope.row.cat_deleted===false"></i>
              <i style="color:red;" class="el-icon-error" v-else></i>
          </template>
          <!-- 排序列 -->
          <template slot="order" slot-scope="scope">
              <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
              <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
              <el-tag size="mini" type="warning" v-else>三级</el-tag>
          </template>
          <!-- 操作列 -->
          <template slot="opt" slot-scope="scope">
              <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          </template>
        </tree-table>
        <!-- 分页区域 -->
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[3, 5, 10, 15]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
        </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%" 
        @close="addCateDialogClosed">
        <!-- 添加分类的表单 -->
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRefs" label-width="100px" >
        <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" >
        
        <!-- options指定数据源 -->
        <!-- props:用来指定配置对象 -->
        <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            
            >
        </el-cascader>
        
        </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
    </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return{
            //查询条件
            queryInfo:{
                type:3,
                pagenum:1,
                pagesize:5
            },
            //商品分类的数据列表，默认为空
            catelist:[],
            //总数据条数
            total:0,
            //为table指定列的定义
            columns:[
                {
                    label:'分类名称',
                    prop:'cat_name'
                },
                {
                    label: '是否有效',
                    prop: 'likes',
                    minWidth: '200px',
                    //表示将当前列定义为模板列
                    type: 'template',
                    //表示当前这一列使用的模板名称
                    template: 'isok',
                },
                 {
                    label:'排序',
                    type:'template',
                    template: 'order',
                },
                {
                    label:'操作',
                    type:'template',
                    template: 'opt',
                }
            ],

            //控制添加分类对话框的显示与隐藏
            addCateDialogVisible:false,
            //添加分类的表单的数据对象
            addCateForm:{
                //将要添加的分类的名称
            cat_name:'',
            //父级分类id
            cat_pid:0,
            //分类的等级默认添加的是一级分类
            cat_level:0
            },
            //添加分类表单的验证规则对象
            addCateFormRules:{
            cat_name:[
                {required:true,message:'请输入分类名称！',trigger:'blur'}
            ]
            },
            //父级分类的列表
            parentCateList:[],
            //指定级联选择器的配置对象
            cascaderProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children',
                checkStrictly:true
            },
            //选中的父级分类的Id数组
            selectedKeys:[]

        }
    },
    created(){
        this.getCateList()
        },
    methods:{
        //获取商品分类数据
        async getCateList(){
           const {data:res}=await this.$http.get('categories',{params:this.queryInfo})
            if(res.meta.status!==200){
                return this.$message.error('获取商品分类失败了！')
            }
            console.log(res.data)
            //把数据列表赋值给castelist
            this.catelist=res.data.result
            //为总数据条数赋值
            this.total=res.data.total
        },
        //监听pageSize 改变
        handleSizeChange(newSize){
            this.queryInfo.pagesize=newSize
            this.getCateList()
        },
        //监听pagenum 改变
        handleCurrentChange(newPage){
            this.queryInfo.pagenum=newPage
            this.getCateList()
        },
        //点击按钮展示添加分类的对话框
        showAddCateDialog(){
            //先获取父级分类的数据列表
            this.getParentCateList()
            //再展示出对话框
            this.addCateDialogVisible=true
            
        },
        //获取父级分类的数据列表
       async getParentCateList(){
           const {data:res}=await this.$http.get('categories',{params:{type:2}})
        if(res.meta.status!==200){
            return this.$message.error('获取父级数据失败了！')
        }
        console.log(res.data)
        this.parentCateList=res.data
        },
        //选择项发生变化就会触发这个函数
        parentCateChanged(){
            console.log(this.selectedKeys)
            //如果selectedKeys 数组中的length 大于0 证明选中了父级分类
            //反之没有选中任何父级分类
            if(this.selectedKeys.length>0){
                //父级分类的Id
              this.addCateForm.cat_pid=  this.selectedKeys[this.selectedKeys.length-1]
             //为当前分类的等级赋值
             this.addCateForm.cat_level=this.selectedKeys.length
             return
          }
          else{
            this.addCateForm.cat_pid=0
             //为当前分类的等级赋值
             this.addCateForm.cat_level=0
             
          }
        },
        //点击确定按钮添加新的分类
        addCate(){
            console.log(this.addCateForm)
            this.$refs.addCateFormRefs.validate(async valid=>{
                if(!valid)return
               const {data:res}=await this.$http.post('categories',this.addCateForm)
               if(res.meta.status!==201){
                   return this.$message.error('添加分类失败了！')
               }
               this.$message.success('添加分类成功了！')   
               this.getCateList()
               this.addCateDialogVisible=false         
          })

        },
        //监听对话框的关闭事件，重置表单数据
        addCateDialogClosed(){
            this.$refs.addCateFormRefs.resetFields()
            this.selectedKeys=[]
            this.addCateForm.cat_level=0
            this.addCateForm.cat_pid=0
        }
    }
}
</script>
<style lang="less" scoped>
.treeTable{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
}
</style>