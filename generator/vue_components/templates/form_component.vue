<template>
  <div class='row'>
  <%_ schema.attributes.forEach((attr) => { _%>
    <b-col lg="6">

      <div
        class='form-group'
        id="fieldset-<%= attr.identifier %>"
      >
        <label class='mb-0' for="<%= attr.identifier %>-input"><%= attr.label %></label>
        <%_ if (attr.required) { _%>
        <span class='text-danger'>*</span>
        <%_ } _%>

        <%_ if (attr.help) { _%>
        <small class='text-muted d-block'>
          <%= attr.help %>
        </small>
        <%_ } _%>

        <!-- <b-form-input id="<%= attr.identifier %>-input" :state="state" v-model="name" trim /> -->

      <%_ if (attr.datatype === DATATYPE_BOOLEAN) { _%>
        <b-form-checkbox v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_STRING) { _%>
        <b-form-input
          trim
          id="<%= attr.identifier %>-input"
          :required="<%= attr.required %>"
          placeholder="<%= attr.label %>"
          v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_STRING_ARRAY) { _%>
        <InputTag
          placeholder="<%= attr.label %>"
          :tags.sync="model.<%=attr.identifier%>"
        />
      <%_ } else if ([DATATYPE_INTEGER, DATATYPE_FLOAT].includes(attr.datatype)) { _%>
        <b-form-input
          type="number"
          id="<%= attr.identifier %>-input"
          :required="<%= attr.required %>"
          placeholder="<%= attr.label %>"
          v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_DATE) { _%>
        <b-form-input
          type="date"
          id="<%= attr.identifier %>-input"
          :required="<%= attr.required %>"
          placeholder="<%= attr.label %>"
          v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_TIME) { _%>
        <b-form-input
          type="time"
          id="<%= attr.identifier %>-input"
          :required="<%= attr.required %>"
          placeholder="<%= attr.label %>"
          v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_DATETIME) { _%>
        <b-form-input
          type="datetime-local"
          id="<%= attr.identifier %>-input"
          :required="<%= attr.required %>"
          placeholder="<%= attr.label %>"
          v-model="model.<%=attr.identifier%>"
        />
      <%_ } else if (attr.datatype === DATATYPE_JSON) { _%>
        <!-- <textarea
          class="form-control"
          placeholder="<%= attr.label %>"
          @change="model.<%=attr.identifier%> = JSON.parse($event.target.value)">{{ JSON.stringify(model.<%=attr.identifier%>, null, 2) }}</textarea> -->
          <p class="text-warning">TODO TODO TODO</p>
          <small>IMPLEMENT JSON EDITING HERE</small>
      <%_ } _%>
      </div>
    </b-col>

    <%_ }) _%>
    <%_ schema.relations.forEach((rel) => { _%>
    <%_ if ([RELATION_TYPE_BELONGS_TO, RELATION_TYPE_HAS_ONE, RELATION_TYPE_HAS_MANY].includes(rel.type)) { _%>
    <div class="col-lg-6">
      <div class="form-group">
        <label class='mb-0'>
          <%= rel.alias.label %>
          <%_ if (rel.required) { %><span class='text-danger'>*</span><% } _%>
        </label>
        <%_ if (rel.type === RELATION_TYPE_BELONGS_TO) { _%>
        <select
          type="text"
          class="form-control"
          placeholder="<%= rel.alias.label %>"
          v-model="model.<%=rel.alias.identifier%>_id"
        >
          <option disabled selected value> -- <%= rel.alias.label_plural %> -- </option>
          <option
            :value="<%=rel.schema.identifier%>._id"
            v-for="<%= rel.schema.identifier %> in <%= rel.alias.identifier_plural %>"
            :key="<%= rel.schema.identifier %>._id"
          >
            {{ <%= rel.schema.identifier %>.<%= rel.related_lead_attribute %> }}
          </option>
        </select>
      <%_ } else if (rel.type === RELATION_TYPE_HAS_MANY) { _%>
        <select
          type="text"
          multiple class="form-control"
          placeholder="<%= rel.alias.label %>"
          v-model="model.<%=rel.alias.identifier%>_ids"
        >
          <option disabled selected value> -- <%= rel.alias.label_plural %> -- </option>
          <option
            :value="<%=rel.schema.identifier%>._id"
            v-for="<%= rel.schema.identifier %> in <%= rel.alias.identifier_plural %>"
            :key="<%= rel.schema.identifier %>._id"
          >
            {{ <%= rel.schema.identifier %>.<%= rel.related_lead_attribute %> }}
          </option>
        </select>
      <%_ } else if (rel.type === RELATION_TYPE_HAS_ONE) { _%>
        <select
          type="text"
          class="form-control"
          placeholder="<%= rel.alias.label %>"
          v-model="model.<%=rel.alias.identifier%>_id"
        >
          <option disabled selected value> -- <%= rel.alias.label_plural %> -- </option>
          <option
            :value="<%=rel.schema.identifier%>._id"
            v-for="<%= rel.schema.identifier %> in <%= rel.alias.identifier_plural %>"
            :key="<%= rel.schema.identifier %>._id"
          >
            {{ <%= rel.schema.identifier %>.<%= rel.related_lead_attribute %> }}
          </option>
        </select>
      <%_ } _%>
      </div>
    </div>

    <%_ } _%>
    <%_ }) _%>
  </div>
</template>

<!-- // // // //  -->

<script>
import InputTag from 'vue-input-tag'

export default {
  name: '<%= schema.class_name %>Form',
  props: {
    model: {
      required: true
    }
  },
  components: {
    InputTag
  },
  created () {
    <%_ schema.relations.forEach((rel) => { _%>
    <%_ if (rel.type !== 'REF_BELONGS_TO') { _%>
    this.$store.dispatch('<%= rel.schema.identifier %>/collection/fetch')
    <%_ } _%>
    <%_ }) _%>
  },
  computed: {
    <%_ let filteredRelations = schema.relations.filter(r => r.type !== 'REF_BELONGS_TO') _%>
    <%_ filteredRelations.forEach((rel, index) => { _%>
    <%= rel.alias.identifier_plural %> () {
      return this.$store.getters['<%= rel.schema.identifier %>/collection/collection']
    }<%= helpers.trailingComma(filteredRelations, index) %>
    <%_ }) _%>
  }
}
</script>
