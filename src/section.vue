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
                    class="contextmenu-right"
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
        <!-- mobile section the v-touch is to vue component that allows gestures on screen  -->
        <v-touch ref="swiper" @swipeleft="nextSlide()" @swiperight="prevSlide()" class="mobile-wrapper">
            <wwObject class="feature-background" :ww-object="section.data.featureBackground" ww-category="background"></wwObject>
            <div class="feature-title" :style="{'border-color': computedFeature.selector.borderColor}">
                <wwContextMenu
                    tag="div"
                    class="contextmenu-right"
                    v-if="editMode"
                    @ww-add-before="addFeature(computedFeatures, currentIndex, 'before')"
                    @ww-add-after="addFeature(computedFeatures, currentIndex, 'after')"
                    @ww-remove="removeFeature(computedFeatures, currentIndex)"
                    @ww-options="customizeColor(computedFeature)"
                >
                    <div class="wwi wwi-config"></div>
                </wwContextMenu>
                <wwLayoutColumn
                    tag="div"
                    ww-default="ww-text"
                    :ww-list="computedFeature.selector.title"
                    @ww-add="add(computedFeature.selector.title, $event)"
                    @ww-remove="remove(computedFeature.selector.title, $event)"
                >
                    <wwObject tag="div" v-for="title in computedFeature.selector.title" :key="title.uniqueId" :ww-object="title"></wwObject>
                </wwLayoutColumn>
            </div>

            <div class="feature-content">
                <wwObject :ww-object="computedFeature.content"></wwObject>
            </div>
        </v-touch>

        <div class="feature-wrapper hidden-mobile" v-for="rootFeature in section.data.rootFeatures" :key="rootFeature.uniqueId">
            <div class="features" v-if="selectedFeature(rootFeature.uniqueId)">
                <wwObject class="feature-background" :ww-object="section.data.featureBackground" ww-category="background"></wwObject>

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

        <div class="content hidden-mobile">
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

//import VueTouch from 'vue-touch';
import Vue from 'vue';

const VueTouch = require('vue-touch')

Vue.use(VueTouch, { name: 'v-touch' })

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            selectedRootFeature: 0,
            selectedFeatureIndex: 0,
            currentIndex: 0
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
        computedFeature() {
            return this.section.data.rootFeatures[this.selectedRootFeature].features[this.currentIndex]
        },
        computedFeatures() {
            return this.section.data.rootFeatures[this.selectedRootFeature].features
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
        if (!this.section.data.contentBackground) {
            needUpdate = true
            this.section.data.contentBackground = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
        }
        if (!this.section.data.featureBackground) {
            needUpdate = true
            this.section.data.featureBackground = wwLib.wwObject.getDefault({
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
    mounted() {
        if (this.editMode)
            this.$refs.swiper.disable('swipe')
    },
    methods: {
        /* wwManager:start */
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);
            this.sectionCtrl.update(this.section);
        },
        /* add root element that contain a new section */
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

        /* add a new section to the slider */
        addFeature(list, _index, where) {
            console.log('list:', list)

            const up = (where == 'after') ? parseInt(1) : 0;
            const index = _index + up
            const newFeature = this.newFeature()

            list.splice(index, 0, newFeature);
            //this.section.data.features.splice(index, 0, newFeature);
            this.sectionCtrl.update(this.section);
        },
        tmpaddFeature(list, _index, where) {
            console.log('list:', list)

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
        /* Open a popup to change the border color */
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
        /* create a popup forum */
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
                                en: 'Choose a border color',
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
        /* basic new columns */
        newColumns() {
            const newColumn = {
                "config": {
                    "count": 2,
                    "xs": {
                        "height": 0,
                        "ignore": false,
                        "cols": [
                            {
                                "offset": 0,
                                "width": 70,
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ],
                                "align": "1",
                                "hide": true
                            },
                            {
                                "offset": "10",
                                "width": "80",
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ],
                                "align": "1"
                            }
                        ],
                        "unit": "%"
                    },
                    "sm": {
                        "ignore": false,
                        "cols": [
                            {
                                "align": "1",
                                "width": "80",
                                "offset": "15",
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ],
                                "hide": true
                            },
                            {
                                "align": "1",
                                "width": "80",
                                "offset": "15",
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ]
                            }
                        ],
                        "height": 0,
                        "unit": "%"
                    },
                    "md": {
                        "ignore": false,
                        "cols": [
                            {
                                "align": "1",
                                "width": "60",
                                "offset": 0,
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ]
                            },
                            {
                                "align": "1",
                                "width": "40",
                                "offset": 0,
                                "borders": [
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    },
                                    {
                                        "color": "#000000",
                                        "style": "solid",
                                        "width": 0
                                    }
                                ]
                            }
                        ],
                        "height": 0,
                        "unit": "%"
                    },
                    "lg": {
                        "ignore": true,
                        "cols": null,
                        "height": 0,
                        "unit": "%"
                    },
                    "height": null
                },
                "columns": [
                    {
                        "background": {
                            "uniqueId": 5205152846,
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
                        "wwObjects": [
                            {
                                "uniqueId": 11768923706,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-columns",
                                    "data": {
                                        "config": {
                                            "count": 1,
                                            "xs": {
                                                "height": 0,
                                                "ignore": false,
                                                "cols": [
                                                    {
                                                        "offset": 0,
                                                        "width": 33.33,
                                                        "borders": [
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            }
                                                        ],
                                                        "align": "1"
                                                    }
                                                ],
                                                "unit": "%"
                                            },
                                            "sm": {
                                                "ignore": false,
                                                "cols": [
                                                    {
                                                        "align": "1",
                                                        "width": "100",
                                                        "offset": 0,
                                                        "borders": [
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            },
                                                            {
                                                                "color": "#000000",
                                                                "style": "solid",
                                                                "width": 0
                                                            }
                                                        ]
                                                    }
                                                ],
                                                "height": 0,
                                                "unit": "%"
                                            },
                                            "md": {
                                                "ignore": true,
                                                "cols": null,
                                                "height": 0,
                                                "unit": "%"
                                            },
                                            "lg": {
                                                "ignore": true,
                                                "cols": null,
                                                "height": 0,
                                                "unit": "%"
                                            },
                                            "height": null
                                        },
                                        "columns": [
                                            {
                                                "background": {
                                                    "uniqueId": 14021993755,
                                                    "link": {
                                                        "data": {},
                                                        "type": "none"
                                                    },
                                                    "content": {
                                                        "data": {
                                                            "style": {
                                                                "boxShadow": {
                                                                    "b": 0,
                                                                    "c": "",
                                                                    "s": 0,
                                                                    "x": 0,
                                                                    "y": 0
                                                                },
                                                                "borderColor": null,
                                                                "borderStyle": null,
                                                                "borderWidth": 0,
                                                                "borderRadius": 0
                                                            },
                                                            "backgroundColor": "#B7EEEEC8",
                                                            "gradient": null
                                                        },
                                                        "type": "ww-color"
                                                    },
                                                    "paddings": {
                                                        "xs": {
                                                            "top": 0,
                                                            "left": 0,
                                                            "right": 0,
                                                            "bottom": 0
                                                        }
                                                    }
                                                },
                                                "wwObjects": [
                                                    {
                                                        "uniqueId": 12656646394,
                                                        "wwVersion": 3,
                                                        "content": {
                                                            "type": "ww-row",
                                                            "data": {
                                                                "wwObjects": [
                                                                    {
                                                                        "uniqueId": 5329801295,
                                                                        "wwVersion": 3,
                                                                        "content": {
                                                                            "type": "ww-text",
                                                                            "data": {
                                                                                "text": {
                                                                                    "fr": "Nouveau texte",
                                                                                    "en": "<span style=\"font-size:1.5rem\"><span style=\"font-size:1.5rem\"><font face=\"Lato, Lato\"><span style=\"font-size:3rem\">Etiam nec suscipit nisl</span></font></span></span>"
                                                                                },
                                                                                "tag": "div",
                                                                                "align": "",
                                                                                "font": "",
                                                                                "size": "",
                                                                                "color": "",
                                                                                "classes": [],
                                                                                "children": []
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
                                                                    }
                                                                ],
                                                                "align": "center",
                                                                "justify": "center"
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
                                                                "top": 25,
                                                                "left": 29,
                                                                "right": 61,
                                                                "bottom": 25
                                                            },
                                                            "sm": {
                                                                "bottom": 0
                                                            },
                                                            "lg": null
                                                        },
                                                        "hidden": false,
                                                        "tags": [],
                                                        "children": {},
                                                        "data": {},
                                                        "anim": {},
                                                        "new": false
                                                    }
                                                ]
                                            }
                                        ]
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
                                        "top": 30,
                                        "left": 85,
                                        "right": 33,
                                        "bottom": 0
                                    },
                                    "sm": null,
                                    "lg": {
                                        "left": 211,
                                        "right": 250,
                                        "top": 65
                                    }
                                },
                                "hidden": false,
                                "tags": [],
                                "children": {},
                                "data": {},
                                "anim": {},
                                "new": false
                            }
                        ]
                    },
                    {
                        "background": {
                            "uniqueId": 9479626160,
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
                        "wwObjects": [
                            {
                                "uniqueId": 4566739628,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-row",
                                    "data": {
                                        "wwObjects": [
                                            {
                                                "uniqueId": 4916000118,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-icon",
                                                    "data": {
                                                        "icon": "wwi wwi-check",
                                                        "style": {
                                                            "color": "#B7EEEEC8",
                                                            "backgroundColor": "#FFFFFF",
                                                            "gradient": {},
                                                            "borderRadius": 50,
                                                            "borderWidth": 2,
                                                            "borderColor": "#B7EEEEC8",
                                                            "borderStyle": "solid",
                                                            "boxShadow": {
                                                                "x": 0,
                                                                "y": 0,
                                                                "b": 0,
                                                                "s": 0,
                                                                "c": ""
                                                            },
                                                            "size": 30,
                                                            "fontSize": 15
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
                                                        "left": 35,
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
                                            {
                                                "uniqueId": 4347174084,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-text",
                                                    "data": {
                                                        "text": {
                                                            "fr": "Nouveau texte",
                                                            "en": "<font face=\"Lato, Lato\"> Aliquam erat volutpat.</font>"
                                                        },
                                                        "tag": "div",
                                                        "align": "",
                                                        "font": "",
                                                        "size": "",
                                                        "color": "",
                                                        "classes": [],
                                                        "children": []
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
                                                        "left": 10,
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
                                            }
                                        ],
                                        "align": "center",
                                        "justify": "flex-start"
                                    }
                                },
                                "link": {
                                    "type": "none",
                                    "data": {}
                                },
                                "ratio": -1,
                                "paddings": {
                                    "xs": {
                                        "top": 20,
                                        "left": 0,
                                        "right": 0,
                                        "bottom": 0
                                    },
                                    "md": {
                                        "top": 38,
                                        "left": 0,
                                        "right": 0,
                                        "bottom": 0
                                    },
                                    "sm": {
                                        "top": 20
                                    },
                                    "lg": null
                                },
                                "hidden": false,
                                "tags": [],
                                "children": {},
                                "data": {},
                                "anim": {},
                                "new": false
                            },
                            {
                                "uniqueId": 14822459284,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-row",
                                    "data": {
                                        "wwObjects": [
                                            {
                                                "uniqueId": 5560042109,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-icon",
                                                    "data": {
                                                        "icon": "wwi wwi-check",
                                                        "style": {
                                                            "color": "#B7EEEEC8",
                                                            "backgroundColor": "#FFFFFF",
                                                            "gradient": {},
                                                            "borderRadius": 50,
                                                            "borderWidth": 2,
                                                            "borderColor": "#B7EEEEC8",
                                                            "borderStyle": "solid",
                                                            "boxShadow": {
                                                                "x": 0,
                                                                "y": 0,
                                                                "b": 0,
                                                                "s": 0,
                                                                "c": ""
                                                            },
                                                            "size": 30,
                                                            "fontSize": 15
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
                                                        "left": 35,
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
                                            {
                                                "uniqueId": 4583407648,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-text",
                                                    "data": {
                                                        "text": {
                                                            "fr": "Nouveau texte",
                                                            "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                        },
                                                        "tag": "div",
                                                        "align": "",
                                                        "font": "",
                                                        "size": "",
                                                        "color": "",
                                                        "classes": [],
                                                        "children": []
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
                                                        "left": 10,
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
                                            }
                                        ],
                                        "align": "center",
                                        "justify": "flex-start"
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
                                        "top": 15,
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
                            {
                                "uniqueId": 10588310731,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-row",
                                    "data": {
                                        "wwObjects": [
                                            {
                                                "uniqueId": 7401733518,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-icon",
                                                    "data": {
                                                        "icon": "wwi wwi-check",
                                                        "style": {
                                                            "color": "#B7EEEEC8",
                                                            "backgroundColor": "#FFFFFF",
                                                            "gradient": {},
                                                            "borderRadius": 50,
                                                            "borderWidth": 2,
                                                            "borderColor": "#B7EEEEC8",
                                                            "borderStyle": "solid",
                                                            "boxShadow": {
                                                                "x": 0,
                                                                "y": 0,
                                                                "b": 0,
                                                                "s": 0,
                                                                "c": ""
                                                            },
                                                            "size": 30,
                                                            "fontSize": 15
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
                                                        "left": 35,
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
                                            {
                                                "uniqueId": 12311132595,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-text",
                                                    "data": {
                                                        "text": {
                                                            "fr": "Nouveau texte",
                                                            "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                        },
                                                        "tag": "div",
                                                        "align": "",
                                                        "font": "",
                                                        "size": "",
                                                        "color": "",
                                                        "classes": [],
                                                        "children": []
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
                                                        "left": 10,
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
                                            }
                                        ],
                                        "align": "center",
                                        "justify": "flex-start"
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
                                        "top": 15,
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
                            {
                                "uniqueId": 9399865954,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-row",
                                    "data": {
                                        "wwObjects": [
                                            {
                                                "uniqueId": 3301531297,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-icon",
                                                    "data": {
                                                        "icon": "wwi wwi-check",
                                                        "style": {
                                                            "color": "#B7EEEEC8",
                                                            "backgroundColor": "#FFFFFF",
                                                            "gradient": {},
                                                            "borderRadius": 50,
                                                            "borderWidth": 2,
                                                            "borderColor": "#B7EEEEC8",
                                                            "borderStyle": "solid",
                                                            "boxShadow": {
                                                                "x": 0,
                                                                "y": 0,
                                                                "b": 0,
                                                                "s": 0,
                                                                "c": ""
                                                            },
                                                            "size": 30,
                                                            "fontSize": 15
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
                                                        "left": 35,
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
                                            {
                                                "uniqueId": 13991075419,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-text",
                                                    "data": {
                                                        "text": {
                                                            "fr": "Nouveau texte",
                                                            "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                        },
                                                        "tag": "div",
                                                        "align": "",
                                                        "font": "",
                                                        "size": "",
                                                        "color": "",
                                                        "classes": [],
                                                        "children": []
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
                                                        "left": 10,
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
                                            }
                                        ],
                                        "align": "center",
                                        "justify": "flex-start"
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
                                        "top": 15,
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
                            {
                                "uniqueId": 12097642231,
                                "wwVersion": 3,
                                "content": {
                                    "type": "ww-row",
                                    "data": {
                                        "wwObjects": [
                                            {
                                                "uniqueId": 11568245663,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-icon",
                                                    "data": {
                                                        "icon": "wwi wwi-check",
                                                        "style": {
                                                            "color": "#B7EEEEC8",
                                                            "backgroundColor": "#FFFFFF",
                                                            "gradient": {},
                                                            "borderRadius": 50,
                                                            "borderWidth": 2,
                                                            "borderColor": "#B7EEEEC8",
                                                            "borderStyle": "solid",
                                                            "boxShadow": {
                                                                "x": 0,
                                                                "y": 0,
                                                                "b": 0,
                                                                "s": 0,
                                                                "c": ""
                                                            },
                                                            "size": 30,
                                                            "fontSize": 15
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
                                                        "left": 35,
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
                                            {
                                                "uniqueId": 13012486766,
                                                "wwVersion": 3,
                                                "content": {
                                                    "type": "ww-text",
                                                    "data": {
                                                        "text": {
                                                            "fr": "Nouveau texte",
                                                            "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                        },
                                                        "tag": "div",
                                                        "align": "",
                                                        "font": "",
                                                        "size": "",
                                                        "color": "",
                                                        "classes": [],
                                                        "children": []
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
                                                        "left": 10,
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
                                            }
                                        ],
                                        "align": "center",
                                        "justify": "flex-start"
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
                                        "top": 15,
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
                            }
                        ]
                    }
                ]
            }
            return newColumn
        },
        newFeature() {
            const newFeature = {
                "selector": {
                    "title": [
                        {
                            "uniqueId": 10278938318,
                            "wwVersion": 3,
                            "content": {
                                "type": "ww-text",
                                "data": {
                                    "text": {
                                        "fr": "Nouveau texte",
                                        "en": "<div style=\"text-align: center;\"><span style=\"font-size: 2rem; font-family: Lato, Lato; text-align: initial;\">Starter</span></div>"
                                    },
                                    "tag": "div",
                                    "align": "",
                                    "font": "",
                                    "size": "",
                                    "color": "",
                                    "classes": [],
                                    "children": []
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
                                    "top": 20,
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
                        {
                            "uniqueId": 9147500319,
                            "wwVersion": 3,
                            "content": {
                                "type": "ww-row",
                                "data": {
                                    "wwObjects": [
                                        {
                                            "uniqueId": 11462546525,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-text",
                                                "data": {
                                                    "text": {
                                                        "fr": "Nouveau texte",
                                                        "en": "<span style=\"font-size:2rem\"><font face=\"Lato, Lato\">$30</font></span>"
                                                    },
                                                    "tag": "div",
                                                    "align": "",
                                                    "font": "",
                                                    "size": "",
                                                    "color": "",
                                                    "classes": [],
                                                    "children": []
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
                                        {
                                            "uniqueId": 8394827038,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-text",
                                                "data": {
                                                    "text": {
                                                        "fr": "Nouveau texte",
                                                        "en": "/month"
                                                    },
                                                    "tag": "div",
                                                    "align": "",
                                                    "font": "",
                                                    "size": "",
                                                    "color": "",
                                                    "classes": [],
                                                    "children": []
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
                                                    "top": 15,
                                                    "left": 5,
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
                                        }
                                    ],
                                    "align": "center",
                                    "justify": "center"
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
                                    "top": 15,
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
                        }
                    ],
                    "borderColor": "#B7EEEEC8"
                },
                "offsetImage": {
                    "uniqueId": 12179570973,
                    "wwVersion": 3,
                    "content": {
                        "type": "ww-image",
                        "data": {
                            "url": "https://cdn.weweb.app/designs/1/sections/Ujq6ZBkxtv2HJZJQEloVYr4JYfJmxUsi.png",
                            "alt": "",
                            "position": {
                                "x": 0,
                                "y": 0
                            },
                            "pos": {
                                "left": "0%",
                                "top": "0%",
                                "width": "100%",
                                "height": "100%"
                            },
                            "zoom": 1.197721179624665,
                            "style": {
                                "borderRadius": 0,
                                "borderWidth": 0,
                                "borderColor": null,
                                "borderStyle": null,
                                "overlay": null,
                                "filter": null,
                                "boxShadow": {
                                    "x": 0,
                                    "y": 0,
                                    "b": 0,
                                    "s": 0,
                                    "c": ""
                                }
                            },
                            "animation": {
                                "duration": 0,
                                "delay": 0,
                                "fn": null,
                                "opacity": 1,
                                "scale": 1,
                                "translateX": 0,
                                "translateY": 0,
                                "rotate": 0
                            },
                            "imgSize": {
                                "w": 1920,
                                "h": 1080
                            },
                            "hover": {
                                "name": "",
                                "options": {}
                            },
                            "crop": "84px100p@8px0p"
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
                            "top": 54,
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
                "content": {
                    "uniqueId": 2733640878,
                    "wwVersion": 3,
                    "content": {
                        "type": "ww-columns",
                        "data": {
                            "config": {
                                "count": 2,
                                "xs": {
                                    "height": 0,
                                    "ignore": false,
                                    "cols": [
                                        {
                                            "offset": 0,
                                            "width": 70,
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ],
                                            "align": "1",
                                            "hide": true
                                        },
                                        {
                                            "offset": "10",
                                            "width": "80",
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ],
                                            "align": "1"
                                        }
                                    ],
                                    "unit": "%"
                                },
                                "sm": {
                                    "ignore": false,
                                    "cols": [
                                        {
                                            "align": "1",
                                            "width": "80",
                                            "offset": "15",
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ],
                                            "hide": true
                                        },
                                        {
                                            "align": "1",
                                            "width": "80",
                                            "offset": "15",
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ]
                                        }
                                    ],
                                    "height": 0,
                                    "unit": "%"
                                },
                                "md": {
                                    "ignore": false,
                                    "cols": [
                                        {
                                            "align": "1",
                                            "width": "60",
                                            "offset": 0,
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ]
                                        },
                                        {
                                            "align": "1",
                                            "width": "40",
                                            "offset": 0,
                                            "borders": [
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                },
                                                {
                                                    "color": "#000000",
                                                    "style": "solid",
                                                    "width": 0
                                                }
                                            ]
                                        }
                                    ],
                                    "height": 0,
                                    "unit": "%"
                                },
                                "lg": {
                                    "ignore": true,
                                    "cols": null,
                                    "height": 0,
                                    "unit": "%"
                                },
                                "height": null
                            },
                            "columns": [
                                {
                                    "background": {
                                        "uniqueId": 5205152846,
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
                                    "wwObjects": [
                                        {
                                            "uniqueId": 11768923706,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-columns",
                                                "data": {
                                                    "config": {
                                                        "count": 1,
                                                        "xs": {
                                                            "height": 0,
                                                            "ignore": false,
                                                            "cols": [
                                                                {
                                                                    "offset": 0,
                                                                    "width": 33.33,
                                                                    "borders": [
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        }
                                                                    ],
                                                                    "align": "1"
                                                                }
                                                            ],
                                                            "unit": "%"
                                                        },
                                                        "sm": {
                                                            "ignore": false,
                                                            "cols": [
                                                                {
                                                                    "align": "1",
                                                                    "width": "100",
                                                                    "offset": 0,
                                                                    "borders": [
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        },
                                                                        {
                                                                            "color": "#000000",
                                                                            "style": "solid",
                                                                            "width": 0
                                                                        }
                                                                    ]
                                                                }
                                                            ],
                                                            "height": 0,
                                                            "unit": "%"
                                                        },
                                                        "md": {
                                                            "ignore": true,
                                                            "cols": null,
                                                            "height": 0,
                                                            "unit": "%"
                                                        },
                                                        "lg": {
                                                            "ignore": true,
                                                            "cols": null,
                                                            "height": 0,
                                                            "unit": "%"
                                                        },
                                                        "height": null
                                                    },
                                                    "columns": [
                                                        {
                                                            "background": {
                                                                "uniqueId": 14021993755,
                                                                "link": {
                                                                    "data": {},
                                                                    "type": "none"
                                                                },
                                                                "content": {
                                                                    "data": {
                                                                        "style": {
                                                                            "boxShadow": {
                                                                                "b": 0,
                                                                                "c": "",
                                                                                "s": 0,
                                                                                "x": 0,
                                                                                "y": 0
                                                                            },
                                                                            "borderColor": null,
                                                                            "borderStyle": null,
                                                                            "borderWidth": 0,
                                                                            "borderRadius": 0
                                                                        },
                                                                        "backgroundColor": "#B7EEEEC8",
                                                                        "gradient": null
                                                                    },
                                                                    "type": "ww-color"
                                                                },
                                                                "paddings": {
                                                                    "xs": {
                                                                        "top": 0,
                                                                        "left": 0,
                                                                        "right": 0,
                                                                        "bottom": 0
                                                                    }
                                                                }
                                                            },
                                                            "wwObjects": [
                                                                {
                                                                    "uniqueId": 12656646394,
                                                                    "wwVersion": 3,
                                                                    "content": {
                                                                        "type": "ww-row",
                                                                        "data": {
                                                                            "wwObjects": [
                                                                                {
                                                                                    "uniqueId": 5329801295,
                                                                                    "wwVersion": 3,
                                                                                    "content": {
                                                                                        "type": "ww-text",
                                                                                        "data": {
                                                                                            "text": {
                                                                                                "fr": "Nouveau texte",
                                                                                                "en": "<span style=\"font-size:1.5rem\"><span style=\"font-size:1.5rem\"><font face=\"Lato, Lato\"><span style=\"font-size:3rem\">Etiam nec suscipit nisl</span></font></span></span>"
                                                                                            },
                                                                                            "tag": "div",
                                                                                            "align": "",
                                                                                            "font": "",
                                                                                            "size": "",
                                                                                            "color": "",
                                                                                            "classes": [],
                                                                                            "children": []
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
                                                                                }
                                                                            ],
                                                                            "align": "center",
                                                                            "justify": "center"
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
                                                                            "top": 25,
                                                                            "left": 29,
                                                                            "right": 61,
                                                                            "bottom": 25
                                                                        },
                                                                        "sm": {
                                                                            "bottom": 0
                                                                        },
                                                                        "lg": null
                                                                    },
                                                                    "hidden": false,
                                                                    "tags": [],
                                                                    "children": {},
                                                                    "data": {},
                                                                    "anim": {},
                                                                    "new": false
                                                                }
                                                            ]
                                                        }
                                                    ]
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
                                                    "top": 30,
                                                    "left": 85,
                                                    "right": 33,
                                                    "bottom": 0
                                                },
                                                "sm": null,
                                                "lg": {
                                                    "left": 211,
                                                    "right": 250,
                                                    "top": 65
                                                }
                                            },
                                            "hidden": false,
                                            "tags": [],
                                            "children": {},
                                            "data": {},
                                            "anim": {},
                                            "new": false
                                        }
                                    ]
                                },
                                {
                                    "background": {
                                        "uniqueId": 9479626160,
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
                                    "wwObjects": [
                                        {
                                            "uniqueId": 4566739628,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-row",
                                                "data": {
                                                    "wwObjects": [
                                                        {
                                                            "uniqueId": 4916000118,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-icon",
                                                                "data": {
                                                                    "icon": "wwi wwi-check",
                                                                    "style": {
                                                                        "color": "#B7EEEEC8",
                                                                        "backgroundColor": "#FFFFFF",
                                                                        "gradient": {},
                                                                        "borderRadius": 50,
                                                                        "borderWidth": 2,
                                                                        "borderColor": "#B7EEEEC8",
                                                                        "borderStyle": "solid",
                                                                        "boxShadow": {
                                                                            "x": 0,
                                                                            "y": 0,
                                                                            "b": 0,
                                                                            "s": 0,
                                                                            "c": ""
                                                                        },
                                                                        "size": 30,
                                                                        "fontSize": 15
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
                                                                    "left": 35,
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
                                                        {
                                                            "uniqueId": 4347174084,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-text",
                                                                "data": {
                                                                    "text": {
                                                                        "fr": "Nouveau texte",
                                                                        "en": "<font face=\"Lato, Lato\"> Aliquam erat volutpat.</font>"
                                                                    },
                                                                    "tag": "div",
                                                                    "align": "",
                                                                    "font": "",
                                                                    "size": "",
                                                                    "color": "",
                                                                    "classes": [],
                                                                    "children": []
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
                                                                    "left": 10,
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
                                                        }
                                                    ],
                                                    "align": "center",
                                                    "justify": "flex-start"
                                                }
                                            },
                                            "link": {
                                                "type": "none",
                                                "data": {}
                                            },
                                            "ratio": -1,
                                            "paddings": {
                                                "xs": {
                                                    "top": 20,
                                                    "left": 0,
                                                    "right": 0,
                                                    "bottom": 0
                                                },
                                                "md": {
                                                    "top": 38,
                                                    "left": 0,
                                                    "right": 0,
                                                    "bottom": 0
                                                },
                                                "sm": {
                                                    "top": 20
                                                },
                                                "lg": null
                                            },
                                            "hidden": false,
                                            "tags": [],
                                            "children": {},
                                            "data": {},
                                            "anim": {},
                                            "new": false
                                        },
                                        {
                                            "uniqueId": 14822459284,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-row",
                                                "data": {
                                                    "wwObjects": [
                                                        {
                                                            "uniqueId": 5560042109,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-icon",
                                                                "data": {
                                                                    "icon": "wwi wwi-check",
                                                                    "style": {
                                                                        "color": "#B7EEEEC8",
                                                                        "backgroundColor": "#FFFFFF",
                                                                        "gradient": {},
                                                                        "borderRadius": 50,
                                                                        "borderWidth": 2,
                                                                        "borderColor": "#B7EEEEC8",
                                                                        "borderStyle": "solid",
                                                                        "boxShadow": {
                                                                            "x": 0,
                                                                            "y": 0,
                                                                            "b": 0,
                                                                            "s": 0,
                                                                            "c": ""
                                                                        },
                                                                        "size": 30,
                                                                        "fontSize": 15
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
                                                                    "left": 35,
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
                                                        {
                                                            "uniqueId": 4583407648,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-text",
                                                                "data": {
                                                                    "text": {
                                                                        "fr": "Nouveau texte",
                                                                        "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                                    },
                                                                    "tag": "div",
                                                                    "align": "",
                                                                    "font": "",
                                                                    "size": "",
                                                                    "color": "",
                                                                    "classes": [],
                                                                    "children": []
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
                                                                    "left": 10,
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
                                                        }
                                                    ],
                                                    "align": "center",
                                                    "justify": "flex-start"
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
                                                    "top": 15,
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
                                        {
                                            "uniqueId": 10588310731,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-row",
                                                "data": {
                                                    "wwObjects": [
                                                        {
                                                            "uniqueId": 7401733518,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-icon",
                                                                "data": {
                                                                    "icon": "wwi wwi-check",
                                                                    "style": {
                                                                        "color": "#B7EEEEC8",
                                                                        "backgroundColor": "#FFFFFF",
                                                                        "gradient": {},
                                                                        "borderRadius": 50,
                                                                        "borderWidth": 2,
                                                                        "borderColor": "#B7EEEEC8",
                                                                        "borderStyle": "solid",
                                                                        "boxShadow": {
                                                                            "x": 0,
                                                                            "y": 0,
                                                                            "b": 0,
                                                                            "s": 0,
                                                                            "c": ""
                                                                        },
                                                                        "size": 30,
                                                                        "fontSize": 15
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
                                                                    "left": 35,
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
                                                        {
                                                            "uniqueId": 12311132595,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-text",
                                                                "data": {
                                                                    "text": {
                                                                        "fr": "Nouveau texte",
                                                                        "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                                    },
                                                                    "tag": "div",
                                                                    "align": "",
                                                                    "font": "",
                                                                    "size": "",
                                                                    "color": "",
                                                                    "classes": [],
                                                                    "children": []
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
                                                                    "left": 10,
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
                                                        }
                                                    ],
                                                    "align": "center",
                                                    "justify": "flex-start"
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
                                                    "top": 15,
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
                                        {
                                            "uniqueId": 9399865954,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-row",
                                                "data": {
                                                    "wwObjects": [
                                                        {
                                                            "uniqueId": 3301531297,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-icon",
                                                                "data": {
                                                                    "icon": "wwi wwi-check",
                                                                    "style": {
                                                                        "color": "#B7EEEEC8",
                                                                        "backgroundColor": "#FFFFFF",
                                                                        "gradient": {},
                                                                        "borderRadius": 50,
                                                                        "borderWidth": 2,
                                                                        "borderColor": "#B7EEEEC8",
                                                                        "borderStyle": "solid",
                                                                        "boxShadow": {
                                                                            "x": 0,
                                                                            "y": 0,
                                                                            "b": 0,
                                                                            "s": 0,
                                                                            "c": ""
                                                                        },
                                                                        "size": 30,
                                                                        "fontSize": 15
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
                                                                    "left": 35,
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
                                                        {
                                                            "uniqueId": 13991075419,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-text",
                                                                "data": {
                                                                    "text": {
                                                                        "fr": "Nouveau texte",
                                                                        "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                                    },
                                                                    "tag": "div",
                                                                    "align": "",
                                                                    "font": "",
                                                                    "size": "",
                                                                    "color": "",
                                                                    "classes": [],
                                                                    "children": []
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
                                                                    "left": 10,
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
                                                        }
                                                    ],
                                                    "align": "center",
                                                    "justify": "flex-start"
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
                                                    "top": 15,
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
                                        {
                                            "uniqueId": 12097642231,
                                            "wwVersion": 3,
                                            "content": {
                                                "type": "ww-row",
                                                "data": {
                                                    "wwObjects": [
                                                        {
                                                            "uniqueId": 11568245663,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-icon",
                                                                "data": {
                                                                    "icon": "wwi wwi-check",
                                                                    "style": {
                                                                        "color": "#B7EEEEC8",
                                                                        "backgroundColor": "#FFFFFF",
                                                                        "gradient": {},
                                                                        "borderRadius": 50,
                                                                        "borderWidth": 2,
                                                                        "borderColor": "#B7EEEEC8",
                                                                        "borderStyle": "solid",
                                                                        "boxShadow": {
                                                                            "x": 0,
                                                                            "y": 0,
                                                                            "b": 0,
                                                                            "s": 0,
                                                                            "c": ""
                                                                        },
                                                                        "size": 30,
                                                                        "fontSize": 15
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
                                                                    "left": 35,
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
                                                        {
                                                            "uniqueId": 13012486766,
                                                            "wwVersion": 3,
                                                            "content": {
                                                                "type": "ww-text",
                                                                "data": {
                                                                    "text": {
                                                                        "fr": "Nouveau texte",
                                                                        "en": "<font face=\"Lato, Lato\">Maecenas sed leo finibus</font><br>"
                                                                    },
                                                                    "tag": "div",
                                                                    "align": "",
                                                                    "font": "",
                                                                    "size": "",
                                                                    "color": "",
                                                                    "classes": [],
                                                                    "children": []
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
                                                                    "left": 10,
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
                                                        }
                                                    ],
                                                    "align": "center",
                                                    "justify": "flex-start"
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
                                                    "top": 15,
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
                                        }
                                    ]
                                }
                            ]
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
                            "bottom": 30
                        },
                        "md": {
                            "top": 43,
                            "left": 0,
                            "right": 0,
                            "bottom": 0
                        },
                        "sm": {},
                        "lg": null
                    },
                    "hidden": false,
                    "tags": [],
                    "children": {},
                    "data": {},
                    "anim": {},
                    "new": false
                }
            }
            wwLib.wwUtils.changeUniqueIds(newFeature)
            return newFeature
        },
        newRootFeature() {
            const rootFeature = {}
            wwLib.wwUtils.changeUniqueIds(rootFeature)
            return rootFeature

        },
        newTempColumns() {
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
        /* Used on mobile version to swipe */
        nextSlide() {
            try {
                let featureLength = this.section.data.rootFeatures[this.selectedRootFeature].features.length - 1

                if (this.currentIndex < featureLength) {
                    ++this.currentIndex
                } else {
                    this.currentIndex = 0
                }
            } catch (err) {
                console.error(err)
            }
        },

        prevSlide() {
            try {
                let featureLength = this.section.data.rootFeatures[this.selectedRootFeature].features.length

                if (this.currentIndex > 0) {
                    --this.currentIndex
                } else {
                    this.currentIndex = featureLength - 1
                }
            } catch (err) {
                console.error(err)
            }

        },

        remove(list, options) {
            console.log('list:', list)
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

        /* update current selected element */
        selectFeature(index) {
            this.selectedFeatureIndex = index
        },
        selectedFeature(id) {
            return id == this.section.data.rootFeatures[this.selectedRootFeature].uniqueId
        },
        selectRootFeature(index) {
            this.selectedRootFeature = index
            this.selectedFeatureIndex = 0
            this.currentIndex = 0
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
.feature-background {
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
        justify-content: center;
        flex-basis: auto;
        min-width: 100px;
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

.feature-wrapper {
    position: relative;
    width: 95%;
    width: 95%;
    margin-left: 2.5%;
    box-shadow: 0 11px 23px -9px rgba(0, 0, 0, 0.5);
    @media (min-width: 992px) {
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 1200px) {
        width: 50%;
        margin-left: 25%;
    }
    .features {
        display: flex;
        justify-content: center;
        flex-wrap: wrap;
        @media (min-width: 768px) {
            min-height: 100px;
            justify-content: space-around;
        }
        .feature {
            position: relative;
            padding: 10px;
            flex-basis: auto;
            border-bottom-width: 2px;
            border-bottom-style: solid;
            cursor: pointer;
            @media (min-width: 768px) {
                min-width: 200px;
            }

            @media (min-width: 1200px) {
                min-width: 300px;
                //flex-basis: 20%;
            }
        }
        .not-selected {
            opacity: 0.4;
            border-bottom: 0px;
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
}

.feature-title {
    position: relative;
    border-bottom-width: 2px;
    border-bottom-style: solid;
    cursor: pointer;
    width: 80%;
    margin-left: 10%;
}

.hidden-mobile {
    display: none;
    @media (min-width: 768px) {
        display: block;
    }
}
.mobile-wrapper {
    display: block;
    position: relative;
    @media (min-width: 768px) {
        display: none;
    }
}

.feature-content {
    margin-bottom: 10%;
}

.contextmenu-right {
    position: absolute;
    top: 0;
    right: 0;
    transform: translate(50%, -50%);
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
.content {
    position: relative;
    background-color: white;
    box-shadow: 0 11px 23px -9px rgba(0, 0, 0, 0.5);
    margin-bottom: 50px;
    width: 95%;
    margin-left: 2.5%;
    min-height: 500px;

    @media (min-width: 992px) {
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 1200px) {
        width: 50%;
        margin-left: 25%;
    }
}
.offset-container {
    width: 100%;
    @media (min-width: 768px) {
        position: absolute;
        transform: translateX(-9%);
        width: 110%;
    }
}

.offset-bg-image {
    display: none;
    @media (min-width: 992px) {
        display: block;
        min-width: 365px;
        min-height: 365px;
        position: absolute;
        transform: translateX(-30%);
    }
}
</style>
