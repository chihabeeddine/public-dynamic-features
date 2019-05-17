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
                    class="contextmenu-left"
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
                    class="contextmenu"
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

            <div class="content-dots-wrapper">
                <li
                    v-for="(dot, index) in computedFeatures"
                    class="content-dot"
                    :style="{'background':computedFeature.selector.borderColor}"
                    :class="{'passive-dot': currentIndex != index}"
                    :key="dot.uniqueId"
                >
                    <div class="dot" @click="switchToIndex(currentIndex, index)"></div>
                </li>
            </div>
        </v-touch>
        <!-- add dots for mobile versionsdn -->
        <!-- Tablet and bigger -->

        <div class="section-wrapper">
            <wwObject class="feature-background" :ww-object="section.data.featureBackground" ww-category="background"></wwObject>
            <div class="feature-wrapper hidden-mobile" v-for="rootFeature in section.data.rootFeatures" :key="rootFeature.uniqueId">
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
                        <wwLayoutColumn
                            tag="div"
                            ww-default="ww-text"
                            :ww-list="feature.selector.title"
                            @ww-add="add(feature.selector.title, $event)"
                            @ww-remove="remove(feature.selector.title, $event)"
                        >
                            <wwObject tag="div" v-for="title in feature.selector.title" :key="title.uniqueId" :ww-object="title"></wwObject>
                        </wwLayoutColumn>
                    </div>
                </div>
            </div>

            <div class="content hidden-mobile">
                <div class="offset-bg-image">
                    <wwObject :ww-object="section.data.rootFeatures[selectedRootFeature].features[selectedFeatureIndex].offsetImage"></wwObject>
                </div>

                <div class="offset-container">
                    <wwObject :ww-object="section.data.rootFeatures[selectedRootFeature].features[selectedFeatureIndex].content"></wwObject>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

import { getNewFeature } from "./defaultFeature"
import { getNewRootFeature } from "./defaultFeature"

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
            this.section.data.rootFeatures = [
                getNewRootFeature()
            ];
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
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newRootFeature = getNewRootFeature()

                this.section.data.rootFeatures.splice(index, 0, newRootFeature);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },

        /* add a new section to the slider */
        addFeature(list, _index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newFeature = getNewFeature()

                list.splice(index, 0, newFeature);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
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
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
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
                wwLib.wwLog.error('ERROR : ', err);
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
                wwLib.wwLog.error('ERROR : ', err);
            }

        },

        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },
        removeFeature(list, index) {
            try {
                this.initPageIndex();
                list.splice(index, 1);
                if (!list.length) {
                    this.addFeature(list, 0, 'after');
                }
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
        },

        removeRootFeature(index) {
            try {
                this.initPageIndex()
                this.section.data.rootFeatures.splice(index, 1);
                if (!this.section.data.rootFeatures.length) {
                    this.addRootFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
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
        },
        switchToIndex(index, position) {
            if (position < this.computedFeatures.length && index != position) {
                this.currentIndex = position
            }
        },
        initPageIndex() {
            this.selectedRootFeature = 0
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
}

.section-wrapper {
    position: relative;
    width: 95%;
    margin-left: 2.5%;
    @media (min-width: 992px) {
        width: 80%;
        margin-left: 10%;
    }
    @media (min-width: 1200px) {
        width: 50%;
        margin-left: 25%;
    }
    .feature-wrapper {
        position: relative;
        width: 95%;
        width: 95%;
        margin-left: 2.5%;
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
                    min-width: 250px;
                }
            }
            .not-selected {
                opacity: 0.4;
                border-bottom: 0px;
            }
        }
    }
    .content {
        position: relative;
        margin-bottom: 50px;
        width: 95%;
        margin-left: 2.5%;
        min-height: 500px;
        .offset-bg-image {
            display: none;
            @media (min-width: 768px) {
                display: block;
                min-width: 365px;
                min-height: 365px;
                position: absolute;
                transform: translateX(-30%);
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
    }
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
    .feature-title {
        position: relative;
        border-bottom-width: 2px;
        border-bottom-style: solid;
        cursor: pointer;
        width: 80%;
        margin-left: 10%;
    }
    .feature-content {
        margin-bottom: 10%;
    }
    .content-dots-wrapper {
        display: flex;
        list-style: none;
        justify-content: center;
        position: relative;
        padding-bottom: 15px;

        .content-dot {
            margin-right: 15px;
            border-radius: 100%;
            .dot {
                cursor: pointer;
                width: 15px;
                height: 15px;
                pointer-events: all;
            }
        }
        .passive-dot {
            opacity: 0.4;
        }
    }
}

/* wwManager:start */
.contextmenu-left {
    position: absolute;
    bottom: 0;
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
</style>
