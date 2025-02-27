<template>
    <div class="menu">
        <div class="container-xxl d-flex flex-column flex-shrink-0 help-trigger"
            :class="{'p-3': !isHorizontal, 'p-h':isHorizontal, 'nav-justified': isHorizontal}"
            :style="styleObjectToString(styles['container'])">

            <div class="help" :style="styleObjectToString(styles['help'])" v-if="help">
                <div class="help-inner" :style="styleObjectToString(styles['help-inner'])">
                    <span class="help-text" :style="styleObjectToString(styles['help-text'])">{{help}}</span>
                </div>
            </div>
            <template v-if="menuTitle">
                <a href="#"
                   class="menu-title align-items-center mb-md-0 me-md-auto text-decoration-none"
                   :style="styleObjectToString(styles['menu-title'])"
                >
                    <i class="icon" :class="menuIcon" :style="styleObjectToString(styles['icon']) + styleObjectToString(styles['menu-icon'])"></i>
                    {{menuTitle}}
                </a>
            <hr>
            </template>
            <ul class="nav nav-pills mb-auto"
               :class="{'flex-column': !isHorizontal, 'nav-justified': isHorizontal}"
               :style="styleObjectToString(styles['nav'])"
            >
                <li class="nav-item" v-for="(option,i) in args.options" :key="option"
                :style="styleObjectToString(styles['nav-item'])"
                >
                    <hr :class="{vr: isHorizontal}" v-if="option === '---'" :style="styleObjectToString(styles['separator'])">
                    <a v-else href="javascript:void(0);"
                       class="nav-link"
                       :class="{active: i == selectedIndex, 'nav-link-horizontal':isHorizontal}"
                       @click="onClicked(i, option)" aria-current="page"
                       :style="styleObjectToString(styles['nav-link']) + styleObjectToString(styles['nav-link-selected'], i == selectedIndex)"
                    >
                        <i class="icon option-icon" :class="[icons[i], i == selectedIndex ? 'option-icon-selected' : '']"
                            :style="styleObjectToString(styles['icon']) + styleObjectToString(styles['option-icon']) + styleObjectToString(styles['option-icon-selected'], i == selectedIndex)"></i>
                        <p class="nav-link-text" :class="[i == selectedIndex ? 'nav-link-text-selected' : '']"
                            :style="styleObjectToString(styles['nav-link-text']) + styleObjectToString(styles['nav-link-text-selected'], i == selectedIndex)">{{option}}</p>
                    </a>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
import { ref, watch } from "vue"
import { Streamlit } from "streamlit-component-lib"
import { useStreamlit } from "./streamlit"
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap-icons/font/bootstrap-icons.css";


function getFullIconName(name) {
    return name.slice(0,3) !== "bi-"? "bi-" + name : name
}

export default {
    name: "MyComponent",
    props: ["args"], // Arguments that are passed to the plugin in Python are accessible in prop "args"
    setup(props) {
        useStreamlit() // lifecycle hooks for automatic Streamlit resize

        // const manualSelect = props.args.manualSelect === undefined || props.args.manualSelect === null ? NaN : props.args.manualSelect;
        const help = ref(props.args.help)
        console.log("HH", help.value)
        const disabled = ref(props.args.disabled)
        const menuTitle = ref(props.args.menuTitle)
        const isHorizontal = props.args.orientation == "horizontal"
        const menuIcon = ref(props.args.menuIcon || "bi-menu-up")
        menuIcon.value = getFullIconName(menuIcon.value)
        const icons = ref(props.args.icons || [])
        for (let i = 0; i < props.args.options.length; i++) {
            if (!icons.value[i]) {
                icons.value[i] = "bi-caret-right";
            }
            icons.value[i] = getFullIconName(icons.value[i]);
        }
        const selectedIndex = ref(props.args.defaultIndex)

        const updateIcons = () => {
        for (let i = 0; i < props.args.options.length; i++) {
            if (!icons.value[i]) {
                icons.value[i] = "bi-caret-right";
            }
            icons.value[i] = getFullIconName(icons.value[i]);
            }
        }
        updateIcons()

        const onClicked = (index, option) => {
            if (disabled.value)
                return
            selectedIndex.value = index
            Streamlit.setComponentValue(option)
        }

        const triggerMenuClick = (index) => {
            console.log("chosen index is: ", index)
            if (index >= 0 && index < props.args.options.length) {
                onClicked(index, props.args.options[index]);
            } else {
                console.warn('Invalid index for triggerMenuClick');
            }
        }

        const styleObjectToString = (obj, condition) => {
            if (typeof condition === "undefined") {
                condition = true
            }
            if (!condition) {
                return ""
            }
            let styleString = ""
            for (const key in obj) {
                styleString += `${key}:${obj[key]};`
            }
            return styleString
        }

        // Get user styles and set up state style
        const disabledCursorStyle = { "cursor": "not-allowed" }
        const lightGreyText = { "color": "rgba(250, 250, 250, 0.4)" }
        const greyedText = { "filter": "invert(40%) sepia(4%);" }
        const fadeBackground = { "filter": "invert(25%) opacity(99%);" }
        const disabledStyles = {
            "nav-item": lightGreyText,
            "nav-link": { ...disabledCursorStyle, ...fadeBackground },
            "nav-link-selected": disabledCursorStyle,
            "nav-link-text": greyedText,
        }

        // Merge the extra styles with users
        const calcStyles = ( ) => {
            const userStyles = props.args.styles || {}
            const extraStyles = disabled.value ? disabledStyles : {}
            const finalStyles = { ...userStyles }
            for (const elementKey in extraStyles) {
                if (!(elementKey in finalStyles))
                    finalStyles[elementKey] = extraStyles[elementKey]

                const cssProps = extraStyles[elementKey]
                for (const [tag, value] of Object.entries(cssProps))
                    finalStyles[elementKey][tag] = value
            }
            // console.log("styles", finalStyles)
            return finalStyles
        }
        const finalStyles = calcStyles()
        const styles = ref(finalStyles);

        watch(
            () => props.args.icons,
            () => {
                // reset icons array and then update it
                icons.value = props.args.icons || []
                updateIcons()
            }
        )

        watch(
            () => props.args.manualSelect,
            (newClickPos, oldClickPos) => {
                if (newClickPos !== undefined && newClickPos !== null && newClickPos !== oldClickPos) {
                    onClicked(newClickPos, props.args.options[newClickPos])
                }
            }
        )

        watch(
            () => props.args.disabled,
            () => {
                disabled.value = props.args.disabled || false
                const updatedStyles = calcStyles()
                styles.value = updatedStyles
            }
        )

        watch(
            () => props.args.help,
            () => {
                help.value = props.args.help || ""
            }
        )

        return {
            triggerMenuClick,
            selectedIndex,
            menuTitle,
            menuIcon,
            icons,
            styles,
            onClicked,
            styleObjectToString,
            isHorizontal,
            help,
            disabled,
        }
    },
}
</script>

<style scoped>
.icon {
    font-size: 1rem;
    margin-right: 0.5rem;
}

.menu hr {
    margin-top: 0.5rem;
    margin-bottom: 0.5rem;
}

.menu .container-xxl {
   background-color: var(--secondary-background-color);
   border-radius: 0.5rem;
}

.menu-title {
    margin-left: 0.75rem;
    margin-right: 0.75rem;
}

.menu-title, .menu-title .icon {
    font-size: 1.5rem;
}

.menu-title .icon {
    margin-right: 0.75rem;
}

.menu-title, .menu .nav-link, .menu .nav-item, hr {
    color: var(--text-color);
}

.nav-link.active {
    color: white;
}

.nav-link:hover {
    background-color: var(--hover-color);
}

.nav-link-hover:hover {
    background-color: inherit;
}

.menu .nav-link {
    /* box-shadow: 0 0px 0.2rem #aaa; */
    margin-top: 0.25rem;
    margin-bottom: 0.25rem;
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
}

.menu .nav-link-horizontal {
    /* margin-top: 0.25rem;
    margin-bottom: 0.25rem; */
    padding-top: 0.25rem;
    padding-bottom: 0.25rem;
}

.p-h {
    padding: 0.5rem 0.75rem;
}

.container .flex-column {
    padding-top: 0.25rem;
}

.menu .nav-item .nav-link.active{
    background-color: var(--primary-color);
}

.nav-link.active, .nav-link.active+.icon {
    font-weight: 900;
}

.vr {
    width: 1px;
    height: 80%;
}

.nav {
    align-items: center;
}

.nav-link {
    display: flex;
    align-items: center;
}

.nav-link-text {
    margin: 0;
}

.help {
    position: absolute; z-index: 1;
    cursor: help;
    width: 33%;
    left: 33%;
    top: 33%;
    z-index: 1;
}

.help-inner {
    visibility: hidden;
    display: flex;

    background-color: black;
    border-radius: 6px;

    height: 36px;
}

.help .help-text {
    user-select: none;
    visibility: hidden;
    width: 100%;
    color: #fff;
    text-align: center;
    padding: 4px 0;
}

.help-trigger:hover .help-text  {
  visibility: visible;
}
.help-trigger:hover .help-inner  {
  visibility: visible;
}
</style>
