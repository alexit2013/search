# search
基于vue+element的查询组件



目前，该查询组件支持选择器（包括级联）、复选框、输入框、日期时间选择器（两种）。可拓展。



使用：

```vue
 <searchCon :searchCon="searchCon" @search="search" ref="search" @searchHeight="searchHeight"></searchCon>
```



```js
data(){
    return{
        isAccords: [
            {id: "1", name: "是"},
            {id: "0", name: "否"}
        ],
        //定义查询内容
        searchCon: {
            selectData: [],
            inputsData: [
                {placeholder: "姓名", name: "name"},
                {placeholder: "电话", name: "phone"},
                {placeholder: "身份证", name: "idCard"},
            ],
            timesData: [
                {
                    startPlaceholder: "开始日期",
                    endPlaceholder: "结束日期",
                    name: ["startTime", "endTime"],
                    type: "daterange",
                    format: "yyyy-MM-dd"
                }
            ]
        },
        searchWords: null,
    }
},
methods:{
    search(obj) {
        this.searchWords = obj;
        //修改分页，重新请求数据
        //this.pageInfo.page = 1;
        //this.$nextTick(() => {
            //this.initList();
        //})
    },
    searchHeight(value){
        console.log(value);
    }
}，
mounted(){
    this.searchCon.selectData.push(
        {
            placeholder: "是否过审",
            name: "isAccord",
            data: this.isAccords
        }
    );
}
```

