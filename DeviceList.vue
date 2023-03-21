<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-cascades"></i> 设备列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <P>
                    网元id：
                    <el-input v-model="search_id" placeholder="请输入网元id" class="handle-input"></el-input>
                    <el-button type="primary" icon="search" @click="search()">搜索</el-button>
                </p>
                <p>
                    <br>
                </p>
                <p>
                    <template>
                        <el-button type="primary" icon="read" class="handle-read" @click="getData">导入数据</el-button>
                        <el-button type="primary" icon="delete" class="handle-del" @click="handleDeleteALL">批量删除</el-button>
                        <el-button type="primary" icon="add" class="handle-add" @click="handleAdd">添加网元</el-button>
                    </template>
                </p>
            </div>
            <el-table :data="tableData" border style="width:100%"
                class="table" 
                ref="multipleTable" 
                total="100"
                page-size="8"
                max-height="350"
                @selection-change="handleSelectionChange">
                <!-- :default-sort = "{prop: 'num', order: 'ascending'}"> -->
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column label="序号" prop="num" type="integer" width="100">
                </el-table-column>
                <el-table-column prop="device_id" label="网元id" type="integer" width="200">
                </el-table-column>
                <el-table-column prop="device_address" label="所属厂区" show-overflow-tooltip>
                </el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.row)">编辑</el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            
            <div class="pagination">
                <el-pagination
                background
                small
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :page-sizes="[5, 10, 15, 20, 30]"
                :page-size="10"
                layout="total, sizes, prev, pager, next, jumper"
                :total="allData.length">
                </el-pagination>
            </div>

            <!-- 分页管理 -->
            <!-- <div class="pagination">
                <el-pagination background @current-change="handleCurrentChange" 
                    layout="prev, pager, next" 
                    :pager-count="10"
                    :total="this.page_num">
                </el-pagination>
            </div> -->
        </div>

        <!-- 添加弹出框 -->
        <el-dialog title="编辑" :visible.sync="addVisible" width="40%">
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <!-- <el-form-item label="序号">
                    <el-input v-model="form.num" type="integer"></el-input>
                </el-form-item> -->
                <el-form-item label="网元id">
                    <el-input v-model="form.device_id" type="integer"></el-input>
                </el-form-item>
                <el-form-item label="所属厂区">
                    <el-input v-model="form.device_address"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAdd">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="40%">
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <!-- <el-form-item label="序号">
                    <el-input v-model="form.num" type="integer"></el-input>
                </el-form-item> -->
                <el-form-item label="网元id">
                    <el-input v-model="form.device_id" type="integer"></el-input>
                </el-form-item>
                <el-form-item label="所属厂区">
                    <el-input v-model="form.device_address"></el-input>
                </el-form-item>

            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
        
        <!-- 搜索弹出框 -->
        <el-dialog title="搜索结果" :visible.sync="searchVisible" width="60%">
            <el-table :data="searchRes">
                <el-table-column label="页号" prop="page_idx" type="integer" width="100">
                </el-table-column>
                <el-table-column label="序号" prop="num" type="integer" width="100">
                </el-table-column>
                <el-table-column prop="device_id" label="网元id" type="integer" width="200">
                </el-table-column>
                <el-table-column prop="device_address" label="所属厂区">
                </el-table-column>
            </el-table>
        </el-dialog>

        <!-- 删除提示框 -->
        <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
            <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="deleteRow">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 批量删除提示框 -->
        <el-dialog title="提示" :visible.sync="delallVisible" width="300px" center>
            <div class="del-all-dialog-cnt">批量删除不可恢复，是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delallVisible = false">取 消</el-button>
                <el-button type="primary" @click="delAll">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'DeviceList',
        data() {
            return {
                url: '/public/vuetable.json',
                allData: [
                {
                    num:'1',
                    device_id:'22',
                    device_address:'撒大大'
                    },
                    {
                    num:'3',
                    device_id:'33',
                    device_address:'所属'
                    },
                    {
                    num:'2',
                    device_id:'11',
                    device_address:'达到'
                    },
                    {
                    num:'5',
                    device_id:'55',
                    device_address:'所属撒旦'
                    },
                    {
                    num:'4',
                    device_id:'44',
                    device_address:'撒旦萨达撒'
                    },
                    {
                    num:'7',
                    device_id:'77',
                    device_address:'撒旦撒旦所属'
                    },
                    {
                    num:'6',
                    device_id:'66',
                    device_address:'撒大苏打大苏打是的'
                    }
                ],
                tableData: [
                ],
                cur_page: 1,
                page_size: 10,
                multipleSelection: [],
                select_cate: '',
                select_word: '',
                search_id: '',
                select_address: '',
                del_list: [],
                is_search: false,
                editVisible: false,
                addVisible: false,
                delVisible: false,
                delallVisible: false,
                searchVisible: false,
                form: {
                    num: '',
                    device_id: '',
                    device_address: ''
                },
                rules: {//还没搞明白怎么限制
                    device_id: [
                        { required: true, message: '请输入网元id', trigger: 'blur' },
                        { type: 'number', message: '设备id必须为数字', trigger: 'blur' }
                    ],
                    device_address: [
                        { required: true, message: '请输入所属厂区', trigger: 'blur' }
                        // { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
                    ]
                },
                idx: -1,
                row: [],
                searchRes: []
            }
        },
        created() {
            // console.log("page",this.cur_page);
            // console.log("pagesize",this.page_size);
            this.getData();
            this.getpageData();
            // console.log("tableData",this.tableData);
        },
        computed: {
            data() {
                return this.tableData.filter((d) => {
                    let is_del = false;
                    for (let i = 0; i < this.del_list.length; i++) {//在删除的列表中寻找表项
                        if (d.device_id === this.del_list[i].device_id) {
                            is_del = true;
                            break;
                        }
                    }
                    if (!is_del) {//删除列表没找到需要的表项
                        if (d.address.indexOf(this.search_id) > -1 &&
                            (d.name.indexOf(this.select_address) > -1 ||
                                d.address.indexOf(this.select_address) > -1)
                        ) {
                            return d;
                        }
                    }
                })
            }
        },
        methods: {
            // 分页导航
            handleSizeChange(val){
                this.page_size = val;
                this.getpageData();
            },
            handleCurrentChange(val) {
                // console.log("val",val);
                // console.log("allData",this.allData);
                this.cur_page = val;
                this.getpageData();
                
            },
            getpageData(){
                // this.sortData();
                this.tableData.splice(0,this.tableData.length);
                for(let i = 0;i < this.page_size; i++){
                    // console.log("idx",(this.cur_page - 1) * this.page_size + i + 1);
                    if(((this.cur_page - 1) * this.page_size + i) < this.allData.length){
                        // console.log("data",this.allData[(this.cur_page - 1) * this.page_size + i]);
                        this.tableData.push(this.allData[(this.cur_page - 1) * this.page_size + i]);
                    }
                    else break;
                }
                for(let i = 0;i < this.tableData.length; i++){
                    this.tableData[i].num = (this.cur_page - 1) * this.page_size + i + 1;
                }
            },
            sortData(){
                for(let i = 0;i < this.allData.length; i++){
                    // console.log("alldataidx",i+1);
                    this.allData[i].num = i + 1;
                }
            },
            // 获取 easy-mock 的模拟数据
            getData() {
                // 开发环境使用 easy-mock 数据，正式环境使用 json 文件
                // if (process.env.NODE_ENV === 'development') {
                //     this.url = '/ms/table/list';
                // };
                // this.$axios.post(this.url).then((res) => {
                //     this.allData = res.data.list;
                //     console.log("tabledata:",this.allData);
                // })
                // this.sortData();
                console.log("getData successfully");
            },
            search() {
                // console.log("search_id",this.search_id);
                this.searchRes.splice(0,this.searchRes.length);
                if(this.search_id){
                    let tmp = {
                        page_idx: '',
                        num: '',
                        device_id: '',
                        device_address: ''
                    };
                    tmp = (this.allData.find((item)=>{
                        if(item.device_id.indexOf(this.search_id) > -1) return item;}))
                    if(tmp){
                        tmp.page_idx = Math.ceil(tmp.num / this.page_size);
                        this.searchRes.push(tmp);
                    }
                    console.log("Res",this.searchRes);
                    this.search_id = '';
                    this.searchVisible = true;
                }
                else{
                    this.$message.error('输入不能为空');
                }
            },
            handleAdd(){
                this.form = {
                    num: '',
                    device_id: '',
                    device_address: ''
                }
                this.addVisible = true;
            },
            handleEdit(row) {
                this.row = row;
                this.idx = this.tableData.indexOf(this.row);
                // console.log("row",this.idx);
                const item = this.tableData[this.idx];
                // console.log("item",item);
                this.form = {
                    num: item.num,
                    device_id: item.device_id,
                    device_address: item.device_address
                }
                // console.log("form",this.form);
                this.editVisible = true;
            },
            handleDelete(row) {
                this.row = row;
                this.delVisible = true;
            },
            handleDeleteALL() {
                // console.log(this.multipleSelection[0].device_id);
                if(this.multipleSelection.length!=0)this.delallVisible = true;
                else this.$message.error('尚未选中要删除行');
            },
            delAll() {//删除选定项
                const length = this.multipleSelection.length;
                this.del_list = this.del_list.concat(this.multipleSelection);
                for (let i = 0; i < length; i++) {
                    this.allData = this.allData.filter((item) => item.device_id !== this.multipleSelection[i].device_id);
                }
                this.getpageData();
                if(this.tableData.length == 0){
                    this.cur_page--;
                    this.getpageData();
                }
                this.multipleSelection = [];
                this.$message.error('批量删除成功');
                this.delallVisible = false;
            },
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            // 保存编辑
            saveEdit() {
                this.$set(this.tableData, this.idx, this.form);
                // console.log("forms",this.form);
                this.editVisible = false;
                this.$message.success(`修改成功`);
            },
            saveAdd() {
                this.form.num = this.allData.length + 1;
                this.allData.push(this.form);
                this.getpageData();
                // console.log("forms",this.form);
                this.addVisible = false;
                this.$message.success(`添加成功`);
            },
            // 确定删除
            deleteRow(){
                this.allData = this.allData.filter((item) => item.device_id !== this.row.device_id);
                this.getpageData();
                if(this.tableData.length == 0){
                    this.cur_page--;
                    this.getpageData();
                }
                this.delVisible = false;
                this.$message.error(`删除成功`);
            }
        }
    }

</script>

<style scoped>
    .handle-box {
        margin-bottom: 20px;
    }

    .handle-select {
        width: 120px;
    }

    .handle-input {
        width: 300px;
        display: inline-block;
    }
    .del-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
    .del-all-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
    .table{
        width: 100%;
        font-size: 14px;
    }
    .red{
        color: #ff0000;
    }
    .mr10{
        margin-right: 10px;
    }
</style>
