vue应用实例记录

1. 实现表单查询功能

   页面方面：

   ```vue
    <div >
             <a-table :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}" rowKey="memberId"  :columns="columns" :dataSource="filteredMemberList" />
           </div>
   ```

   JS方面，computed中监控`this.memberDataList`和`this.searchString`，当这两个变量改变 时，`filteredMemberList`就会跟着改变，重新计算后返回到table中

   ```js
    computed: {          
           filteredMemberList(){
             let members_array = this.memberDataList;
             if(this.searchString == ''){
               return members_array;
             }
             members_array = this.memberDataList.filter(item => {
               return item.memberName.match(this.searchString)
             });
             // 返回过来后的数组
             return members_array;;
           },        
           
         },
   ```

   

2. 实列

