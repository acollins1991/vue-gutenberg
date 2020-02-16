<template>
  <div>
    <div
      v-for="component, index in contentComponents"
      v-bind:key="'display' + index"
      v-html="component.content"
    ></div>
    <div v-for="component, index in contentComponents" v-bind:key="'input' + index">
      <input
        type="text"
        v-model.lazy="component.content"
        v-on:input="updateContentComponent($event.target.value, $event.target.dataset)"
        v-bind:data-component="JSON.stringify(component)"
      />
    </div>
    <br />
    <textarea rows="10" v-model="pageData.content"></textarea>
  </div>
</template>

<script>
const initialPageData = require("../assets/InitialValue.json");

export default {
  name: "PageInput",
  data: function() {
    return {
      pageData: initialPageData
    };
  },
  computed: {
    contentComponents: function() {
      let contentComponents = [];
      // split content after each closing pb comment
      const componentRegex = /(<!-- open-pb:(.*?) -->(.*?)<!-- close-pb:(.*?) -->)/gm;
      const pageContent = this.pageData.content;
      const pageContentSplit = pageContent.match(componentRegex);
      // create component array from component content strings
      pageContentSplit.forEach(string => {
        // get json data from component comment
        // TODO: improve regex to stop relying on index matching
        let componentJsonData = string.match(/(<!-- open-pb:(.*?) -->)/);
        componentJsonData = JSON.parse(componentJsonData[2]);
        // get content from component
        // TODO: improve regex to stop relying on index matching
        let componentContent = string.match(
          /(<!-- open-pb:(.*?) -->(.*?)<!-- close-pb:(.*?) -->)/
        );
        componentJsonData.content = componentContent[3];

        contentComponents.push(componentJsonData);
      });

      return contentComponents;
    }
  },
  methods: {
    updateContentComponent(newValue, dataSet) {
      let componentData = JSON.parse(dataSet.component);
      // get content before altering object
      let componentContent = componentData.content;
      // remove content from object
      delete componentData.content;
      // created escaped data string
      let componentDataString = JSON.stringify(componentData);
      componentDataString = componentDataString.replace(/"/g, '"');
      let newComponentContentString =
        "<!-- open-pb:" +
        componentDataString +
        " -->" +
        componentContent +
        "<!-- close-pb:" +
        componentData.component +
        " -->";
      // find and replace component string with regex
      // TODO: improve reliability of this as likely only to work once, as stringifying process removes spaces
      let regex = new RegExp(
        '<!-- open-pb:(.*"id": ' +
          componentData.id +
          ".*) -->(.*?)<!-- close-pb:(.*?) -->"
      );
      this.pageData.content = this.pageData.content.replace(
        regex,
        newComponentContentString
      );
      console.log(newComponentContentString);
    }
  }
};
</script>