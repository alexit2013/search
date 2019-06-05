<template>
    <template class="rightSelect" ref="select">
        <template v-if="showArea">
            <el-select v-model="province" clearable placeholder="请选择省">
                <el-option v-for="(ele,i) in provinces" :key="i" :label="ele.name" :value="ele.code"></el-option>
            </el-select>
            <template v-if="isHiddenCity !== 'true' ">
                <el-select v-model="city" clearable placeholder="请选择市" :disabled="!province">
                    <el-option v-for="(ele,i) in citys" :key="i" :label="ele.name" :value="ele.code"></el-option>
                </el-select>
            </template>
            <template v-if="isHiddenArea !== 'true'">
                <el-select v-model="area" clearable placeholder="请选择区" :disabled="!city">
                    <el-option v-for="(ele,i) in areas" :key="i" :label="ele.name" :value="ele.code"></el-option>
                </el-select>
            </template>
            <template v-if="isHiddenStreet !== 'true'">
                <el-select v-model="street" clearable placeholder="请选择商圈" :disabled="!area">
                    <el-option v-for="(ele,i) in streets" :key="i" :label="ele.name" :value="ele.code"></el-option>
                </el-select>
            </template>
        </template>
        <template v-for="(item,index) in searchCon.selectData">
            <el-select
                    v-if="!item.isRemote && !item.isCascader"
                    :key="index"
                    v-model="item.value"
                    clearable
                    @change="change"
                    :placeholder="'请选择'+item.placeholder">
                <el-option
                        v-if="item.data"
                        v-for="(e,i) in item.data"
                        :key="i"
                        :label="e.name"
                        :value="e.id">
                </el-option>
            </el-select>
            <template v-else>
                <el-select
                        :key="index"
                        v-if="item.isRemote"
                        filterable
                        remote
                        v-model="item.value"
                        :remote-method="getRemoteUserList(item,index)"
                        clearable
                        @change="change"
                        :placeholder="'请选择'+item.placeholder">
                    <el-option
                            v-if="item.data"
                            v-for="(e,i) in remoteList[index]"
                            :key="i"
                            :label="e.name"
                            :value="e.id">
                    </el-option>
                </el-select>
                <el-cascader
                        :key="index"
                        v-else-if="item.isCascader"
                        clearable
                        filterable
                        :placeholder="'请选择'+item.placeholder"
                        change-on-select
                        :options="item.data"
                        :props="item.props"
                        v-model="item.value">
                </el-cascader>
            </template>
        </template>
        <el-input
                v-for="(item,index) in searchCon.inputsData"
                :key="item.placeholder+index"
                v-model="item.value"
                :placeholder="'请输入'+item.placeholder"
                clearable>
        </el-input>
        <template v-for="(item,index) in searchCon.timesData">
            <el-date-picker
                    :key="item.placeholder+index"
                    v-if="!item.type || item.type === 'date' || item.type === 'datetime'"
                    v-model="item.value"
                    :type="item.type" :placeholder="'请选择'+item.placeholder"
                    :value-format="item.format ? item.format : 'yyyy-MM-dd HH:mm:ss'"
                    clearable
                    class="dataCon"
                    @change="checkTime(item.name)">
            </el-date-picker>
            <el-date-picker
                    :key="item.placeholder+index"
                    v-if="item.type === 'daterange' || item.type === 'datetimerange'"
                    clearable
                    class="dataCon"
                    v-model="item.value"
                    :type="item.type"
                    range-separator="-"
                    :value-format="item.format ? item.format : 'yyyy-MM-dd HH:mm:ss'"
                    :start-placeholder="'请选择'+item.startPlaceholder"
                    :end-placeholder="'请选择'+item.endPlaceholder"
                    @change="change">
            </el-date-picker>
        </template>
        <template v-for="(item,index) in searchCon.checkBoxData">
            <el-checkbox v-model="item.value" :key="index">{{item.placeholder}}</el-checkbox>
        </template>
        <slot></slot>
        <el-button class="newBtn searchBtn" type="primary" @click="searchAll" icon="el-icon-search"></el-button>
        <!--<el-button class="oldBtn" @click="clearAll">清空</el-button>-->
    </template>
</template>

<script>
    export default {
        name: "searchCon",
        props: {
            searchCon: {
                type: Object,
                default: function () {
                    return {
                        selectData: [
                            {
                                //非必填
                                placeholder: "",
                                //查询返回的字段名，必填
                                name: "",
                                //下拉列表数据，必填
                                //数据格式为id、name
                                data: [
                                    {
                                        id: "id",
                                        name: "name",
                                    }
                                ],
                                //选择框中的值默认为空，非必填
                                value: "" || [],
                                //是否远程搜索，默认false
                                isRemote: false,
                                //远程搜索接口
                                remoteApi: Function,
                                //是否级联搜索，默认false
                                isCascader: false,
                                //级联选择器接收的参数除了children，只有value和label，用于定义数据标识字段
                                props: {
                                    value: "id",
                                    label: "title"
                                }
                            }
                        ],
                        inputsData: [
                            {
                                //必填
                                placeholder: "",
                                name: "",
                                value: ""
                            }
                        ],
                        timesData: [
                            {    //日期时间插件默认为date类型，还可指定datetime类型，
                                // 类型为'daterange'或'datetimerange'时，需指定startPlaceholder和endPlaceholder
                                //使用：
                                // {
                                    // startPlaceholder: "开始日期",
                                    // endPlaceholder: "结束日期",
                                    // name: ["startTime", "endTime"],
                                    // type: "daterange"
                                // }
                                //非必填
                                type: "date",
                                //日期时间插件指定value-format类型---详见element
                                //非必填
                                format: "yyyy-MM-dd HH:mm:ss",
                                startPlaceholder: "",
                                endPlaceholder: "",
                                placeholder: "",
                                name: "" || [],
                                value: "" || [],
                                //rule为function时回调验证规则
                                //rule为array时触发当前定义的一条或多条验证规则
                                //非必填
                                rule: Function || Array
                            }
                        ],
                    }
                }
            },
            //结束时间大于开始时间的验证规则
            timeRuleType: {
                type: Object,
                default: function () {
                    return {
                        //是否不需要验证结束时间大于开始时间，默认为true
                        noCheck: false,
                        //指定开始时间的字段为beginTime，必填
                        beginTime: "beginTime",
                        //必填
                        endTime: "endTime",
                    }
                }
            },
            //是否显示地区,，默认为false
            showArea: {
                type: Boolean,
                default: false
            },
            //不显示城市，默认为false
            isHiddenCity: {
                type: Boolean,
                default: false
            },
            //不显示地区，默认为false
            isHiddenArea: {
                type: Boolean,
                default: false
            },
            //不显示商圈，默认为false
            isHiddenStreet: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                provinces: [],
                citys: [],
                areas: [],
                streets: [],
                //最终的地区代码
                areaCode: "",
                province: "",
                area: "",
                city: "",
                street: "",
                //远程搜索列表
                remoteList: [],
            }
        },
        methods: {
            change(){
                //强制刷新,因为数据层次太多，render函数没有自动更新，需手动强制刷新。
                this.$forceUpdate()
            },
            getRemoteUserList(item, index) {
                console.log(item)
                this.remoteList[index] = [];
                item.remoteApi(item.value).then(res => {
                    this.remoteList[index] = res;
                })
            },
            //清空
            clearAll() {
                for (let item in this.searchCon) {
                    this.searchCon[item].forEach((ele, i) => {
                        if (ele.value instanceof Array) {
                            this.$set(this.searchCon[item], i, {...ele, value: []})
                        } else if (ele.value instanceof Boolean) {
                            let length = ele.value.length;
                            let value = length.reduce((result, item) => {
                                return result.push(false);
                            }, [])
                            this.$set(this.searchCon[item], i, {...ele, value})
                        } else {
                            this.$set(this.searchCon[item], i, {...ele, value: ""})
                        }
                    });
                }

                if (this.showArea) {
                    this.areaCode = "";
                    this.province = "";
                }

                this.$emit('search', {});
            },
            searchAll() {
                let obj = {};
                if (this.searchCon.selectData && this.searchCon.selectData.length) {
                    this.searchCon.selectData.forEach(ele => {
                        //当前是级联选择器时
                        if (ele.value instanceof Array) {
                            obj[ele.name] = ele.value[ele.value.length - 1];
                        } else {
                            obj[ele.name] = ele.value;
                        }
                    });
                }
                if (this.searchCon.inputsData && this.searchCon.inputsData.length) {
                    this.searchCon.inputsData.forEach(ele => {
                        obj[ele.name] = ele.value;
                    });
                }
                if (this.searchCon.timesData && this.searchCon.timesData.length) {
                    this.searchCon.timesData.forEach(ele => {
                        //当前事件类型是datetimerange或daterange时
                        if ((ele.value instanceof Array) && (ele.name instanceof Array)) {
                           ele.name.forEach(e => {
                               ele.value.forEach(i => {
                                   obj[e] = i;
                               })
                           })
                        } else {
                            obj[ele.name] = ele.value;
                        }
                    });
                }

                if (this.showArea) {
                    this.areaCode = this.street || this.area || this.city || this.province || "";
                    obj.areaCode = this.areaCode;
                }
                this.$emit('search', obj);
            },
            //////////////////////////////////////时间验证//////////////////////////////////////////////
            //用户确认选定的值时触发时间验证
            checkTime(name) {
                this.searchCon.timesData.forEach((ele, index) => {
                    //验证当前值
                    if (ele.name == name) {
                        if (ele.rule) {
                            let type = (typeof ele.rule).toLowerCase();
                            //自定义验证---回调
                            if (type === 'function') {
                                ele.rule(ele.value, (msg) => {
                                    if (msg) {
                                        ele.value = "";
                                        this.$message.warning(msg);
                                    }
                                });
                                //当前的多条验证规则
                            } else if (type === 'object') {
                                ele.rule.forEach(e => {
                                    //验证当前时间，ele.value当前值，index当前值对应的下标
                                    this['timeRule' + e](ele.value, index);
                                })
                            }
                        }
                        //需要验证结束时间 并且 当前是结束时间时触发
                        if (!this.timeRuleType.noCheck && ele.name === this.timeRuleType.endTime) {
                            this.getTimeInfo();
                        }
                    }
                });
                this.$forceUpdate()
            },
            //获取开始时间、结束时间的值和位置
            getTimeInfo() {
                let item = this.timeRuleType;
                let beginTime, endTime, beginIndex, endIndex;
                this.searchCon.timesData.forEach((ele, i) => {
                    if (item.beginTime && ele.name === item.beginTime) {
                        beginTime = ele.value;
                        beginIndex = i;
                    }
                    if (item.endTime && ele.name === item.endTime) {
                        endTime = ele.value;
                        endIndex = i;
                    }
                });
                this.timeRule(beginTime, endTime, beginIndex, endIndex);
            },
            timeRule(beginTime, endTime, beginIndex, endIndex) {
                if (new Date(endTime).getTime() < new Date(beginTime).getTime() - 8.64e7) {
                    this.$message.warning('结束时间不能小于开始时间!');
                    this.searchCon.timesData[endIndex].value = "";
                }
            },
            timeRule1(value, index) {
                if (new Date(value).getTime() > Date.now()) {
                    this.$message.warning('开始时间不能大于今天!');
                    this.searchCon.timesData[index].value = "";
                }
            },
            ///////////////////////////////////////地区////////////////////////////////////////////////
            request(code, callback) {
                areaTree({code}).then(res => {
                    callback(res.data);
                });
            },
        },
        mounted(){
            this.$nextTick(() => {
                this.$emit('searchHeight', this.$refs.select.clientHeight);
            })

            if (this.showArea) {
                this.request(0, (data) => {
                    this.provinces = data;
                })
            }
        },
        updated(){
          console.log(this.$refs.select.clientHeight)
            this.$nextTick(() => {
                console.log(this.$refs.select.clientHeight)
                this.$emit('searchHeight', this.$refs.select.clientHeight);
            })
        },
        watch: {
            province() {
                if (!this.showArea) return;
                this.city = "";
                this.area = "";
                this.street = "";
                if (this.province) {
                    if (this.isHiddenCity === 'true') return;
                    this.request(this.province, (data) => {
                        this.citys = data;
                    })
                } else {
                    this.citys = [];
                    this.areas = [];
                    this.streets = [];
                }
            },
            city() {
                this.area = "";
                this.street = "";
                if (this.city) {
                    if (this.isHiddenArea === 'true') return;
                    this.request(this.city, (data) => {
                        this.areas = data;
                    })
                } else {
                    this.areas = [];
                    this.streets = [];
                }
            },
            area() {
                this.street = "";
                if (this.area) {
                    if (this.isHiddenStreet === 'true') return;
                    this.request(this.area, (data) => {
                        this.streets = data;
                    })
                } else {
                    this.streets = [];
                }
            },
        }
    }
</script>

<style scoped lang="scss">
    .rightSelect {
        /deep/ {
            .el-input__inner {
                height: 30px;
                line-height: 30px;
                font-size: 12px;
            }
            .el-input__icon {
                line-height: 30px;
                font-size: 12px;
            }
            i {
                font-size: 14px;
            }
            .el-select, .el-input, .el-cascader, .el-checkbox {
                margin-right: $padding-8;
                margin-bottom: $padding-8;
                width: auto;
            }
            .el-button {
                //padding: 6px $padding-16;
                margin-bottom: $padding-8;
            }
            .el-range-editor.el-input__inner {
                padding: 0;
                margin-bottom: $padding-8;
                margin-right: $padding-8;
                .el-range__icon {
                    margin-left: 0;
                }
            }
            .el-date-editor .el-range-separator {
                font-size: $font-12;
                line-height: 30px;
            }
        }
    }
</style>
