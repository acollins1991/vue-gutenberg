<template>
    <div>
        <div v-for="component, index in contentComponents" v-bind:key="index" v-html="component.content"></div>
        <textarea rows="10" v-model="pageDate.content"></textarea>
        <div v-for="component, index in contentComponents" v-bind:key="index">
            <input type="text" v-model="component.content">
        </div>
    </div>
</template>

<script>
const initialPageData = require('../assets/InitialValue.json');

export default {
    name: "PageInput",
    data: function() {
        return {
            pageDate: initialPageData
        }
    },
    computed: {
        contentComponents: () => {
            let contentComponents = [];
            // split content after each closing pb comment
            const componentRegex = /(<!-- open-pb:(.*?) -->(.*?)<!-- close-pb:(.*?) -->)/gm;
            const pageContent = initialPageData.content;
            const pageContentSplit = pageContent.match(componentRegex);
            // create component array from component content strings
            pageContentSplit.forEach( string => {
                // get json data from component comment
                // TODO: improve regex to stop relying on index matching
                let componentJsonData = string.match(/(<!-- open-pb:(.*?) -->)/);
                componentJsonData = JSON.parse(componentJsonData[2]);
                // get content from component
                // TODO: improve regex to stop relying on index matching
                let componentContent = string.match(/(<!-- open-pb:(.*?) -->(.*?)<!-- close-pb:(.*?) -->)/);
                componentJsonData.content = componentContent[3];
                
                contentComponents.push(componentJsonData);
            });

            return contentComponents;

        }
    }
}
</script>