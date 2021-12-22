<template>
  <v-text-field
      ref="input"
      :label="label"
      :value="iMaskInstance ? iMaskInstance.value : value"
      @blur="setLazy"
  />
</template>

<script>
import IMask from 'imask';

export default {
  name: 'VMaskedTextField',
  props: {
    mask: {
      type: Object,
      required: true
    },
    value: {
      type: String,
    },
    label: {
      type: String,
    },
  },
  data: () => ({
    $el: null,
    $elData: null,
    iMaskInstance: null,
  }),
  mounted() {
    this.initMask()
  },
  methods: {
    setLazy() {
      if (this.$elData.lazyValue !== this.iMaskInstance.value) {
        this.$elData.lazyValue = this.iMaskInstance.value;
      }
    },
    initMask() {
      this.$el = this.$refs.input.$el.querySelector('input');
      this.$elData = this.$refs.input._data;
      this.iMaskInstance = IMask(this.$el, this.mask)
          .on('accept', this.onAccept.bind(this));
    },
    onAccept() {
      const maskValue = this.iMaskInstance.value;
      this.$emit('input', this.unmaskValue(maskValue));
    },
    updateValue () {
      this.iMaskInstance.value = this.value == null ? '' : this.value;
    },
    maskValue () {
      return this.iMaskInstance.value;
    },
    unmaskValue(maskValue) {
      const maskName = this.mask?.mask?.name;
      if (maskName && maskName.includes('Date')) {
      // return the date masked otherwise the date mask will not work correctly when removing characters.
        return maskValue;
      }

      return IMask.pipe(
          maskValue,
          this.mask,
          IMask.PIPE_TYPE.MASKED,
          IMask.PIPE_TYPE.UNMASKED
      );
    },
    destroyMask () {
      if (this.iMaskInstance) {
        this.iMaskInstance.destroy();
        delete this.iMaskInstance;
      }
    }
  },
  watch: {
    $props: {
      handler(props) {
        if (this.mask) {
          if (this.iMaskInstance) {
            this.iMaskInstance.updateOptions(this.mask);
            if ('value' in props &&
                (props.value !== this.maskValue() ||
                    (typeof props.value !== 'string' && this.iMaskInstance.value === '') &&
                    !this.iMaskInstance.el.isActive)
            ) {
              this.updateValue();
            }
          } else {
            this.initMask();
            if (props.value !== this.maskValue()) this.onAccept();
          }
        } else {
          this.destroyMask();
          if ('value' in props) this.$el.value = props.value;
        }
      },
      deep: true
    }
  }
}
</script>

