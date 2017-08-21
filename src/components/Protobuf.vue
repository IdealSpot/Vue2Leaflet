<template>
</template>

<script>

import eventsBinder from '../utils/eventsBinder.js';
import propsBinder from '../utils/propsBinder.js';

const events = [
  'click',
  'dblclick',
  'mousedown',
  'mouseover',
  'mouseout',
  'contextmenu',
  'loading',
  'tileunload',
  'tileloadstart',
  'tileerror',
  'tileload',
  'load',
  'add',
  'remove',
  'popupopen',
  'popupclose',
  'tooltipopen',
  'tooltipclose'
]

const props = {
  url: String,
  attribution: {
    type: String,
    custom: true
  },
  token: {
    type: String,
    custom: true
  },
  interactive: {
    type: Boolean,
    custom: true,
    default: function() {
      return false;
    }
  },
  visible: {
    type: Boolean,
    custom: true,
    default: true,
  },
  restyle: {
    type: Boolean,
    custom: true
  },
  styleFunction: {
    type: Function,
    custom: true
  },
  params: {
    type: Object,
    custom: true,
    default: function() {
      return {};
    }
  }
};

export default {
  props: props,
  mounted() {
    if (this.attribution) this.params['attribution'] = this.attribution;
    if (this.token) this.params['token'] = this.token;
    this.params['interactive'] = this.interactive;
    this.mapObject = L.vectorGrid.protobuf(this.url, this.params);
    eventsBinder(this, this.mapObject, events);
    propsBinder(this, this.mapObject, props);
  },
  beforeDestroy() {
    this.setVisible(false);
  },
  methods: {
    deferredMountedTo(parent) {
      this.parent = parent;
      this.mapObject.addTo(parent);
      this.attributionControl = parent.attributionControl;
      var that = this.mapObject;
      for (var i = 0; i < this.$children.length; i++) {
        this.$children[i].deferredMountedTo(that);
      }
    },
    setAttribution(val, old) {
      this.attributionControl.removeAttribution(old);
      this.attributionControl.addAttribution(val);
    },
    setToken(val) {
      this.params.token = val;
    },
    setInteractive(val) {
      this.params.interactive = val;
    },
    setParams(val) {
      if (newVal == oldVal) return;
      if (this.mapObject) {
        if (newVal) {
          this.parent.removeLayer(this.mapObject);
          this.mapObject = null;
          this.mapObject = L.vectorGrid.protobuf(this.url, this.params);
          this.mapObject.addTo(this.parent);
        }
      }
    },
    setRestyle(val) {
      if (!val || !this.styleFunction) return;
      this.styleFunction(this.mapObject);
      this.$emit('l-restyled', true);
    },
    setStyleFunction(val) {
      if (!val) return;
      this.styleFunction(this.mapObject);
      this.$emit('l-restyled', true);
    },
    setVisible(newVal, oldVal) {
      if (newVal == oldVal) return;
      if (this.mapObject) {
        if (newVal) {
          this.mapObject.addTo(this.parent);
        } else {
          this.parent.removeLayer(this.mapObject);
        }
      }
    }
  }
};
</script>
