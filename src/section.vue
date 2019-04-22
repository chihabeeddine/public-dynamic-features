<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div>
        <!-- wwManager:start -->
        <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- Weweb Wallpaper -->
        <wwObject class="background" v-bind:ww-object="section.data.bg" ww-category="background"></wwObject>

        <div class="features">
            <div class="feature" v-for="(feature, index) in section.data.features" :key="feature.selector.uniqueId" @click="selectFeature(index)">
                <wwContextMenu tag="div" class="contextmenu" v-if="editMode" @ww-add-before="addFeature(index, 'before')" @ww-add-after="addFeature(index, 'after')" @ww-remove="removeFeature(index)">
                    <div class="wwi wwi-config"></div>
                </wwContextMenu>
                <wwObject v-bind:ww-object="feature.selector.image" :class="{'selected-feature-image': index == selectedFeatureIndex}"></wwObject>
                <wwObject v-bind:ww-object="feature.selector.label" :class="{'selected-feature-label': index == selectedFeatureIndex}"></wwObject>
            </div>
        </div>
        <div class="content">
            <wwObject v-bind:ww-object="section.data.features[selectedFeatureIndex].content"></wwObject>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            selectedFeatureIndex: 0
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        }
    },
    created() {

        let needUpdate = false

        //Initialize section data
        this.section.data = this.section.data || {};

        if (!this.section.data.bg) {
            needUpdate = true
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
        }

        if (!this.section.data.features || this.section.data.features == 0) {
            needUpdate = true
            this.section.data.features = [{
                selector: {
                    image: wwLib.wwObject.getDefault({
                        type: 'ww-image',
                        data: {
                            url: 'https://cdn.weweb.app/public/images/no_image_selected.png'
                        }
                    }),
                    label: wwLib.wwObject.getDefault({ type: 'ww-text' })
                },
                content: wwLib.wwObject.getDefault({
                    type: 'ww-columns'
                })
            }];
        }

        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }

        this.selectNextfeatureInterval = setInterval(() => {
            if (this.editMode) return;
            if (this.section.data.features.length - 1 == this.selectedFeatureIndex) {
                this.selectedFeatureIndex = 0
            } else {
                this.selectedFeatureIndex++
            }
        }, 3000);


    },
    methods: {
        /* wwManager:start */
        addFeature(_index, where) {
            const up = (where == 'after') ? parseInt(1) : 0;
            const index = _index + up
            const newFeature = {
                selector: {
                    image: wwLib.wwObject.getDefault({
                        type: 'ww-image',
                        data: {
                            url: 'https://cdn.weweb.app/public/images/no_image_selected.png'
                        }
                    }),
                    label: wwLib.wwObject.getDefault({ type: 'ww-text' })
                },
                content: wwLib.wwObject.getDefault({
                    type: 'ww-columns',
                    "data": {
                        "config": {
                            "count": 1,
                            "xs": {
                                "height": null,
                                "ignore": false,
                                "cols": [
                                    {
                                        "offset": 0,
                                        "width": 33.33,
                                        "borders": [],
                                        "align": "1"
                                    },
                                    {
                                        "offset": 0,
                                        "width": 33.33,
                                        "borders": [],
                                        "align": "1"
                                    },
                                    {
                                        "offset": 0,
                                        "width": 33.33,
                                        "borders": [],
                                        "align": "1"
                                    }
                                ]
                            },
                            "sm": {
                                "ignore": true,
                                "cols": []
                            },
                            "md": {
                                "ignore": true,
                                "cols": []
                            },
                            "lg": {
                                "ignore": true,
                                "cols": []
                            },
                            "height": null
                        },
                        "columns": [
                            {
                                "background": {
                                    "uniqueId": 11452749013,
                                    "wwVersion": 3,
                                    "content": {
                                        "type": "ww-color",
                                        "data": {
                                            "backgroundColor": "transparent",
                                            "style": {
                                                "borderRadius": 0,
                                                "borderWidth": 0,
                                                "borderColor": null,
                                                "borderStyle": null,
                                                "boxShadow": {
                                                    "x": 0,
                                                    "y": 0,
                                                    "b": 0,
                                                    "s": 0,
                                                    "c": ""
                                                }
                                            }
                                        }
                                    },
                                    "link": {
                                        "type": "none",
                                        "data": {}
                                    },
                                    "ratio": -1,
                                    "paddings": {
                                        "xs": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        },
                                        "md": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        }
                                    },
                                    "hidden": false,
                                    "tags": [],
                                    "children": {},
                                    "data": {},
                                    "anim": {},
                                    "new": false
                                },
                                "wwObjects": []
                            },
                            {
                                "background": {
                                    "uniqueId": 5073082716,
                                    "wwVersion": 3,
                                    "content": {
                                        "type": "ww-color",
                                        "data": {
                                            "backgroundColor": "transparent",
                                            "style": {
                                                "borderRadius": 0,
                                                "borderWidth": 0,
                                                "borderColor": null,
                                                "borderStyle": null,
                                                "boxShadow": {
                                                    "x": 0,
                                                    "y": 0,
                                                    "b": 0,
                                                    "s": 0,
                                                    "c": ""
                                                }
                                            }
                                        }
                                    },
                                    "link": {
                                        "type": "none",
                                        "data": {}
                                    },
                                    "ratio": -1,
                                    "paddings": {
                                        "xs": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        },
                                        "md": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        }
                                    },
                                    "hidden": false,
                                    "tags": [],
                                    "children": {},
                                    "data": {},
                                    "anim": {},
                                    "new": false
                                },
                                "wwObjects": []
                            },
                            {
                                "background": {
                                    "uniqueId": 2089555755,
                                    "wwVersion": 3,
                                    "content": {
                                        "type": "ww-color",
                                        "data": {
                                            "backgroundColor": "transparent",
                                            "style": {
                                                "borderRadius": 0,
                                                "borderWidth": 0,
                                                "borderColor": null,
                                                "borderStyle": null,
                                                "boxShadow": {
                                                    "x": 0,
                                                    "y": 0,
                                                    "b": 0,
                                                    "s": 0,
                                                    "c": ""
                                                }
                                            }
                                        }
                                    },
                                    "link": {
                                        "type": "none",
                                        "data": {}
                                    },
                                    "ratio": -1,
                                    "paddings": {
                                        "xs": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        },
                                        "md": {
                                            "top": 0,
                                            "left": 0,
                                            "right": 0,
                                            "bottom": 0
                                        }
                                    },
                                    "hidden": false,
                                    "tags": [],
                                    "children": {},
                                    "data": {},
                                    "anim": {},
                                    "new": false
                                },
                                "wwObjects": []
                            }
                        ]
                    }
                })
            }
            this.section.data.features.splice(index, 0, newFeature);
            this.sectionCtrl.update(this.section);
        },
        removeFeature(index) {
            this.section.data.features.splice(index, 1);
            if (!this.section.data.features.length) {
                this.addFeature(0, 'after');
            }
            this.sectionCtrl.update(this.section);
        },
        /* wwManager:end */
        selectFeature(index) {
            clearInterval(this.selectNextfeatureInterval);
            this.selectedFeatureIndex = index
        }
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}

.features {
    display: flex;
    justify-content: center;
    width: 90%;
    margin-left: 5%;
    margin-top: 50px;
    @media (min-width: 768px) {
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 992px) {
        width: 60%;
        margin-left: 20%;
    }
    @media (min-width: 1200px) {
        width: 50%;
        margin-left: 25%;
    }
    .feature {
        position: relative;
        width: 200px;
        margin-right: 20px;
        cursor: pointer;
    }
    .selected-feature-image {
        border-color: blue;
    }
    .selected-feature-label {
        color: #ce003b;
    }
    /* wwManager:start */
    .contextmenu {
        position: absolute;
        top: 0;
        left: 0;
        transform: translate(-50%, -50%);
        width: 30px;
        height: 30px;
        color: white;
        background-color: #ef811a;
        border-radius: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 1.2rem;
        cursor: pointer;
        z-index: 1;
    }
    /* wwManager:end */
}
.content {
    position: relative;
    background-color: white;
    box-shadow: 0 11px 23px -9px rgba(0, 0, 0, 0.5);
    margin: 50px 0;
    width: 95%;
    margin-left: 2.5%;
    @media (min-width: 768px) {
        margin: 50px 0;
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 992px) {
        margin: 50px 0;
        width: 75%;
        margin-left: 12.5%;
    }
    @media (min-width: 1200px) {
        margin: 50px 0;
        width: 70%;
        margin-left: 15%;
    }
}
</style>
