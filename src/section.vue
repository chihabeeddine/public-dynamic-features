<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div>
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>

        <div class="root-features">
            <div class="root-feature" v-for="(rootFeature, index) in section.data.rootFeatures" :key="rootFeature.uniqueId" @click="selectRootFeature(index)">
                <wwContextMenu
                    tag="div"
                    class="contextmenu"
                    v-if="editMode"
                    @ww-add-before="addRootFeature(index, 'before')"
                    @ww-add-after="addRootFeature(index, 'after')"
                    @ww-remove="removeRootFeature(index)"
                >
                    <div class="wwi wwi-config"></div>
                </wwContextMenu>

                <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="rootFeature.rootTitle" @ww-add="add(rootFeature.rootTitle, $event)" @ww-remove="remove(rootFeature.rootTitle, $event)">
                    <wwObject tag="div" v-for="title in rootFeature.rootTitle" :key="title.uniqueId" :ww-object="title"></wwObject>
                </wwLayoutColumn>
            </div>
        </div>

        <div v-for="rootFeature in section.data.rootFeatures" :key="rootFeature.uniqueId">
            <div class="features" v-if="selectedFeature(rootFeature.uniqueId)">
                <div
                    class="feature"
                    v-for="(feature, index) in rootFeature.features"
                    :key="feature.selector.uniqueId"
                    @click="selectFeature(index)"
                    :class="{'not-selected': index != selectedFeatureIndex}"
                    :style="{'border-color': feature.selector.borderColor}"
                >
                    <wwContextMenu
                        tag="div"
                        class="contextmenu"
                        v-if="editMode"
                        @ww-add-before="addFeature(rootFeature.features, index, 'before')"
                        @ww-add-after="addFeature(rootFeature.features, index, 'after')"
                        @ww-remove="removeFeature(rootFeature.features, index)"
                        @ww-options="customizeColor(feature)"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="feature.selector.title" @ww-add="add(feature.selector.title, $event)" @ww-remove="remove(feature.selector.title, $event)">
                        <wwObject tag="div" v-for="title in feature.selector.title" :key="title.uniqueId" :ww-object="title"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
        </div>

        <div class="content">
            <wwObject class="background" :ww-object="section.data.contentBackground" ww-category="background"></wwObject>
            <div class="offset-bg-image">
                <wwObject :ww-object="section.data.rootFeatures[selectedRootFeature].features[selectedFeatureIndex].offsetImage"></wwObject>
            </div>

            <div class="offset-container">
                <wwObject :ww-object="section.data.rootFeatures[selectedRootFeature].features[selectedFeatureIndex].content"></wwObject>
            </div>
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
            selectedRootFeature: 0,
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
        },

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
        if (!this.section.data.contentBackground) {
            needUpdate = true
            this.section.data.contentBackground = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
        }
        if (!this.section.data.offsetContentImage) {
            needUpdate = true
            this.section.data.offsetContentImage = wwLib.wwObject.getDefault({
                type: 'ww-image',
                data: {
                    url: 'https://cdn.weweb.app/designs/1/sections/Ujq6ZBkxtv2HJZJQEloVYr4JYfJmxUsi.png'
                }
            });
        }

        if (!this.section.data.rootFeatures) {
            needUpdate = true
            this.section.data.rootFeatures = [{
                rootTitle: [
                    wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: "E-shop",
                                fr: "E-commerce"
                            }
                        }
                    })
                ],
                features: [{
                    selector: {
                        title: [
                            wwLib.wwObject.getDefault({
                                type: 'ww-text',
                                data: {
                                    text: {
                                        en: "Starter",
                                        fr: "Starter"
                                    }
                                }
                            })
                        ],
                        borderColor: "#575174"
                    },
                    offsetImage: wwLib.wwObject.getDefault({
                        type: 'ww-image',
                        data: {
                            url: 'https://cdn.weweb.app/designs/1/sections/Ujq6ZBkxtv2HJZJQEloVYr4JYfJmxUsi.png'
                        }
                    }),
                    content: wwLib.wwObject.getDefault({
                        type: 'ww-columns',
                        data: this.newColumns()
                    })
                }],
                uniqueId: wwLib.wwUtils.getUniqueId()
            }];
        }
        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }
    },
    methods: {
        /* wwManager:start */
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);
            this.sectionCtrl.update(this.section);
        },
        addRootFeature(_index, where) {
            const up = (where == 'after') ? parseInt(1) : 0;
            const index = _index + up
            const newRootFeature = {
                rootTitle: [
                    wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: "E-shop",
                                fr: "E-commerce"
                            }
                        }
                    })
                ],
                features: [{
                    selector: {
                        title: [
                            wwLib.wwObject.getDefault({
                                type: 'ww-text',
                                data: {
                                    text: {
                                        en: "Starter",
                                        fr: "Starter"
                                    }
                                }
                            })
                        ],
                        borderColor: "#575174"
                    },
                    offsetImage: wwLib.wwObject.getDefault({
                        type: 'ww-image',
                        data: {
                            url: 'https://cdn.weweb.app/designs/1/sections/Ujq6ZBkxtv2HJZJQEloVYr4JYfJmxUsi.png'
                        }
                    }),
                    content: wwLib.wwObject.getDefault({
                        type: 'ww-columns',
                        data: this.newColumns()
                    })
                }],
                uniqueId: wwLib.wwUtils.getUniqueId()
            }
            //splice the new feature
            this.section.data.rootFeatures.splice(index, 0, newRootFeature);
            this.sectionCtrl.update(this.section);
        },

        addFeature(list, _index, where) {
            const up = (where == 'after') ? parseInt(1) : 0;
            const index = _index + up
            const newFeature = {
                selector: {
                    title: [
                        wwLib.wwObject.getDefault({
                            type: 'ww-text',
                            data: {
                                text: {
                                    en: "Starter",
                                    fr: "Starter"
                                }
                            }
                        })
                    ],
                    borderColor: "#575174"
                },
                offsetImage: wwLib.wwObject.getDefault({
                    type: 'ww-image',
                    data: {
                        url: 'https://cdn.weweb.app/designs/1/sections/Ujq6ZBkxtv2HJZJQEloVYr4JYfJmxUsi.png'
                    }
                }),
                content: wwLib.wwObject.getDefault({
                    type: 'ww-columns',
                    data: this.newColumns()
                })
            }

            list.splice(index, 0, newFeature);
            //this.section.data.features.splice(index, 0, newFeature);
            this.sectionCtrl.update(this.section);
        },
        async customizeColor(feature) {
            try {
                wwLib.wwObjectHover.setLock(this);

                let options = await this.edit()
                const result = await wwLib.wwPopups.open(options);
                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                if (typeof (result) != 'undefined') {
                    if (typeof (result.borderColor) != 'undefined') {
                        feature.selector.borderColor = result.borderColor
                    }
                    this.sectionCtrl.update(this.section);
                }
                wwLib.wwObjectHover.removeLock();
            } catch (error) {
                console.error(error);
            }
        },
        async edit() {


            wwLib.wwPopups.addStory('WWTIP_CUSTOM', {
                title: {
                    en: 'Color picker',
                    fr: 'Choisir une couleur'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Border Color:',
                                fr: 'Couleur de la bordure :'
                            },
                            type: 'color',
                            key: 'borderColor',
                            value: "#42b983",
                            valueData: 'borderColor',
                            desc: {
                                en: 'Choose a border color for the block',
                                fr: 'Changer la couleur de la bordure'
                            }
                        },

                    ]
                },
                buttons: {

                    NEXT: {
                        text: {
                            en: 'Ok',
                            fr: 'Ok'
                        },
                        next: false
                    }
                }
            })



            let options = {
                firstPage: 'WWTIP_CUSTOM',
                data: {
                    wwObject: this.wwObject,
                }
            }

            return options
        },
        newColumns() {
            const newColumn = {
                "config": {
                    "count": 2,
                    "xs": {
                        "height": null,
                        "ignore": false,
                        "cols": [
                            {
                                "offset": 0,
                                "width": 70,
                                "borders": [],
                                "align": "1"
                            },
                            {
                                "offset": 0,
                                "width": 30,
                                "borders": [],
                                "align": "1"
                            },

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
                            "uniqueId": wwLib.wwUtils.getUniqueId(),
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
                            "uniqueId": wwLib.wwUtils.getUniqueId(),
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

                ]
            }
            return newColumn
        },

        remove(list, options) {
            list.splice(options.index, 1);
            this.sectionCtrl.update(this.section);
        },
        removeFeature(list, index) {
            list.splice(index, 1);
            if (!list.length) {
                this.addFeature(0, 'after');
            }
            this.sectionCtrl.update(this.section);
        },
        removeRootFeature(index) {
            this.section.data.rootFeatures.splice(index, 1);
            if (!this.section.data.rootFeatures.length) {
                this.addRootFeature(0, 'after');
            }
            this.sectionCtrl.update(this.section);
        },
        /* wwManager:end */
        selectFeature(index) {
            this.selectedFeatureIndex = index

        },
        selectedFeature(id) {
            return id == this.section.data.rootFeatures[this.selectedRootFeature].uniqueId
        },
        selectRootFeature(index) {
            this.selectedRootFeature = index
            this.selectedFeatureIndex = 0
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

.root-features {
    display: flex;
    justify-content: center;
    width: 95%;
    margin-left: 2.5%;
    margin-top: 100px;
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
    .root-feature {
        position: relative;
        padding: 10px;
        justify-content: center;
        flex-basis: auto;
        min-width: 100px;
        margin-right: 50px;
        cursor: pointer;
        @media (min-width: 768px) {
            //flex-basis: 20%;
        }
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

.features {
    display: flex;
    justify-content: center;
    width: 95%;
    margin-left: 2.5%;
    background-color: white;
    box-shadow: 0 11px 23px -9px rgba(0, 0, 0, 0.5);
    margin-top: 50px;
    flex-wrap: wrap;
    @media (min-width: 768px) {
        justify-content: space-around;
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 992px) {
        width: 75%;
        margin-left: 12.5%;
    }
    @media (min-width: 1200px) {
        width: 70%;
        margin-left: 15%;
    }
    .feature {
        position: relative;
        padding: 10px;
        flex-basis: auto;
        margin-right: 50px;
        border-bottom-width: 2px;
        border-bottom-style: solid;
        //border-bottom: 2px solid #5e3de8;
        cursor: pointer;
        @media (min-width: 768px) {
            min-width: 150px;
            //flex-basis: 20%;
        }
    }
    .not-selected {
        opacity: 0.6;
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
    margin-bottom: 50px;
    width: 95%;
    margin-left: 2.5%;
    min-height: 500px;
    @media (min-width: 768px) {
        margin-bottom: 50px;
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 992px) {
        margin-bottom: 50px;
        width: 75%;
        margin-left: 12.5%;
    }
    @media (min-width: 1200px) {
        margin-bottom: 50px;
        width: 70%;
        margin-left: 15%;
    }
}
.offset-container {
    @media (min-width: 768px) {
        position: absolute;
        transform: translateX(-9%);
        width: 110%;
    }
}

.offset-bg-image {
    min-width: 365px;
    min-height: 365px;
    position: absolute;
    transform: translateX(-30%);
}
</style>
