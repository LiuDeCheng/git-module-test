<template>
    <div>
        <el-form-item label-width="0">
            <el-divider class="custom-divider">{{ i18nt('designer.setting.topicAssociation') }}</el-divider>
        </el-form-item>
        <el-form-item :label="i18nt('designer.setting.topicAssociation')">
            <el-button type="primary" size="mini" @click="addRules">添加规则</el-button>
            <div class="numberOfRules">规则数量：{{ optionModel.topicAssociation.length }}条</div>
        </el-form-item>
        <el-dialog :title="i18nt('designer.setting.topicAssociation') + '添加规则'" :visible.sync="rulesShow"
                   append-to-body :close-on-click-modal="false" :close-on-press-escape="false" width="70%">
            <div class="label">当前题目：{{ optionModel.label }}</div>
            <el-card class="box-card">
                <div>显示当前题目的条件为：</div>
                <div v-for="(item, index) in ruleList" :key="item.key"
                     style="margin-top: 12px;display: flex;align-items: center;">
                    <el-select v-model="item.relation" placeholder="关系"
                               :style="{'width': '100px', 'visibility': index === 0 ? 'hidden': 'visible'}" size="mini">
                        <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                    <el-select v-model="item.subjectId" placeholder="请选择题目" style="width: 280px" size="mini"
                               @change="subjectChange($event, item)">
                        <el-option
                                v-for="item in subjectList"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                    <el-select v-model="item.matchRule" placeholder="筛选" style="width: 160px" size="mini">
                        <el-option
                                v-for="item in screenOptions"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                    <el-select v-model="item.subjectOptions" clearable multiple placeholder="请选择选项值"
                               style="width: 280px" size="mini" v-show="!!item.subjectId">
                        <el-option
                                v-for="item in item.options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                    <i class="el-icon-remove-outline"
                       style="display: inline-block;margin-left: 12px;font-size: 16px"
                       @click="removeRuleItem(index)"></i>
                </div>
                <el-button type="primary" size="mini" style="margin-top: 12px" @click="addRuleItem">新增规则</el-button>
            </el-card>
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="okRule">确 定</el-button>
              </span>
        </el-dialog>
    </div>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
    name: "topicAssociation-editor",
    mixins: [i18n],
    props: {
        designer: Object,
        selectedWidget: Object,
        optionModel: Object,
    },
    data() {
        return {
            // 弹窗空值
            rulesShow: false,
            // 可进行关联的属性
            relationTypeArray: ['select', 'radio', 'checkbox'],
            // 筛选选项
            screenOptions: [
                {
                    value: 'any',
                    label: '选择任一选项'
                }, {
                    value: 'all',
                    label: '选择全部选项'
                }, {
                    value: 'noAny',
                    label: '没有选择任一'
                }, {
                    value: 'noAll',
                    label: '没有选择全部'
                }
            ],
            // 关系
            options: [{
                value: 'and',
                label: '且'
            }, {
                value: 'or',
                label: '或'
            }],
            // 题目列表
            subjectList: [],
            // 规则列表
            ruleList: [], // [{relation: 关系（且、或）, subjectId: 题目ID, matchRule: 'any\all\noAny\noAll', subjectOptions: [选项值,选项值,...]}]
        }
    },
    created() {
        console.log('optionModel', JSON.parse(JSON.stringify(this.optionModel)))
        console.log('designer', this.designer.widgetList)
        console.log('selectedWidget', this.selectedWidget)
    },
    methods: {
        // 添加规则-弹窗
        addRules() {
            this.subjectFormat()
            this.rulesShow = true
        },
        // 添加规则行
        addRuleItem() {
            this.ruleList.push({
                relation: 'and',
                subjectId: '',
                matchRule: 'any',
                subjectOptions: [],
                options: [],
                key: this.randomNumber()
            })
        },
        // 移除规则行
        removeRuleItem(index) {
            this.ruleList.splice(index, 1)
        },
        // 题目切换
        subjectChange(e, item) {
            item.options = []
            item.subjectOptions = []
            if (e) {
                const find = this.designer.widgetList.find(j => j.id === e)
                console.log('find', find.options.optionItems)
                item.options = find ? find.options.optionItems : []
            }
        },
        // ok
        okRule() {
            let msg = ''
            for (let i in this.ruleList) {
                const {subjectId, subjectOptions} = this.ruleList[i]
                if (!subjectId) {
                    msg = '请选择题目'
                    break
                }
                if (!subjectOptions || subjectOptions.length === 0) {
                    msg = '请选择选项值'
                    break
                }
            }
            if (msg) {
                this.$message.error(msg)
                return
            }
            console.log(this.ruleList)
            this.$emit('change', {
                data: this.ruleList.map(i => {
                    return {
                        relation: i.relation,
                        subjectId: i.subjectId,
                        matchRule: i.matchRule,
                        subjectOptions: i.subjectOptions
                    }
                }), key: 'topicAssociation'
            })
            this.rulesShow = false
        },
        // 随机值
        randomNumber(){
            let res = "";
            for(let i=0;i<10;i++){
                res += Math.floor(Math.random()*10);
            }
            return res;
        },
        // 题目处理-type = "select"\"radio"\"checkbox""的时候进行插入
        subjectFormat() {
            console.log('widgetList', JSON.parse(JSON.stringify(this.designer.widgetList)))
            console.log('optionModel', JSON.parse(JSON.stringify(this.optionModel)))
            console.log('selectedWidget', JSON.parse(JSON.stringify(this.selectedWidget)))
            this.ruleList = this.selectedWidget.options.topicAssociation.map(i => {
                const find = this.designer.widgetList.find(j => j.id === i.subjectId)
                i['options'] = find.options.optionItems || []
                i['key'] = this.randomNumber()
                return i
            })
            if (this.ruleList.length === 0) {
                this.ruleList = [{
                    relation: 'and',
                    subjectId: '',
                    matchRule: 'any',
                    subjectOptions: [],
                    options: [],
                    key: this.randomNumber()
                }]
            }
            console.log('this.ruleList', this.ruleList)
            const list = []
            this.designer.widgetList.forEach(item => {
                if (this.relationTypeArray.includes(item.type) && item.id !== this.optionModel.name) {
                    list.push({
                        label: item.options.label,
                        value: item.id
                    })
                }
            })
            this.subjectList = list
        }
    }
}
</script>

<style scoped>
.numberOfRules {
    font-size: 12px;
}

.label {
    font-weight: 600;
}
</style>
