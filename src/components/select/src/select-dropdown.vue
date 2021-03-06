<template>
    <div class="am-selected-content am-dropdown-content" v-if="visible">
        <div class="am-selected-search" v-if="search" @click.stop>
            <am-input-group :block="true" size="sm">
                <am-input-label :transparent="true" slot="prepend"><am-icon type="search" :color="color" ></am-icon></am-input-label>
                <am-input v-model="searchVal" placeholder="请输入搜索内容" size="sm"></am-input>
            </am-input-group>
        </div>
        <ul class="am-selected-list" ref="lists">
            <li
                v-for="item,key in renderOptions"
                :class="{'am-checked': isSelected(item)}"
                :key="key"
                @click.stop="selectHandle(item, key)"
            >
                <span class="am-selected-text">{{ item.label }}</span>
                <i class="am-icon-check"></i>
            </li>
        </ul>
    </div>
</template>

<script>
    import Popup from '../../../mixins/popup';
    import * as doms from '../../../utils/dom';
    import { Input, InputGroup, InputLabel } from '../../input/';
    import { Icon } from '../../icon/';

    export default {
        name: 'am-select-dropdown',
        mixins: [ Popup ],
        data() {
            const selectValue = [];
            this.options.forEach((item) => {
                if (!this.multiple && selectValue.length) {
                    return ;
                }
                if (item.selected) {
                    selectValue.push(item)
                }
            });
            return {
                selectValue,
                saveOptions: this.options,
                renderOptions: this.options,
                searchVal: '',
            }
        },
        props: [ 'value', 'options', 'maxHeight', 'isFoucs', 'search', 'multiple', 'width', 'color' ],
        methods: {
            isSelected(item) {
                let is = false;
                this.selectValue.every((_item) => {
                    if (_item.value === item.value && _item.label === item.label) {
                        is = true;
                        return false;
                    }
                    return true;
                });
                return is;
            },
            removeSelectItem(item) {
                this.selectValue.splice(this.selectValue.findIndex((_item) => {
                    return _item.label === item.label && _item.value === item.value;
                }), 1);
            },
            selectHandle(item, key) {
                if (this.isSelected(item)) {
                    if (!this.multiple) {
                        return this.hide();
                    }
                    else {
                        return this.removeSelectItem(item);
                    }
                }

                if (!this.multiple) {
                    this.selectValue = [item];
                    this.hide();
                }
                else {
                    this.selectValue.push(item);
                }
            }
        },
        watch: {
            isFoucs(curVal, oldVal) {
                if (curVal) {
                    this.show();
                }
                else {
                    this.hide();
                }
            },
            options(curVal, oldVal) {
                curVal.forEach((item) => {
                    if (this.renderOptions.findIndex((_item) => {
                        return _item.label === item.label && _item.value === item.value;
                    }) > -1) {
                        this.renderOptions.push(item);
                        this.saveOptions.push(item);
                        if (item.selected) {
                            if (this.multiple) {
                                this.selectValue.push(item);
                            }
                            else {
                                this.selectValue = [item];
                            }
                        }
                    }
                });
            },
            selectValue(curVal, oldVal) {
                this.$emit('input', curVal);
            },
            visible(curVal, oldVal) {
                if (curVal) {
                    this.$parent.isFoucs = true;
                }
                else {
                    this.$parent.isFoucs = false;
                }
            },
            searchVal(curVal, oldVal) {
                if (!curVal) {
                    this.renderOptions = this.saveOptions;
                }
                else {
                    const filter = [];
                    this.renderOptions.forEach((item) => {
                        if (item.label.indexOf(curVal) > -1 || item.guesser.indexOf(curVal) > -1) {
                            filter.push(item);
                        }
                    });
                    this.renderOptions = filter;
                }
            }
        },
        updated() {
            if (this.visible) {
                const { top, left, height, width } = this.$parent.$el.getBoundingClientRect();
                const { top: offsetTop, left: offsetLeft } =  this.getPageOffset();
                const cssProperty = {
                    top: top + offsetTop + height + 'px',
                    left: left + offsetLeft + 'px',
                    zIndex: this.getZIndex()
                };
                if (this.width !== undefined) {
                    if (this.width.indexOf('%') > 0) {
                        cssProperty['width'] = parseInt(this.width, 10) / 100 * width + 'px';
                    }
                    else {
                        cssProperty['width'] = this.width;
                    }
                }
                doms.css(this.$el, cssProperty);
                if (this.maxHeight !== undefined) {
                    doms.css(this.$refs['lists'], {
                        'maxHeight': this.maxHeight,
                        'overflowY': 'scroll'
                    });
                }
            }
        },
        mounted() {
            document.body.appendChild(this.$el);
        },
        components: {
            AmInput: Input,
            AmIcon: Icon,
            AmInputGroup: InputGroup,
            AmInputLabel: InputLabel
        }
    }
</script>