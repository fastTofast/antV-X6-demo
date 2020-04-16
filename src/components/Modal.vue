<template>
  <transition name="modal">
    <div class="cus-modal" v-if="show">
      <div class="close" @click="close">
        <i class="el-icon-close"></i>
      </div>
      <div class="header">
        <slot name="header" class="tset">{{ title }}</slot>
      </div>
      <div class="content">
        <slot name="content"></slot>
      </div>
      <div class="footer">
        <slot name="footer"></slot>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    show: {
      required: true,
      type: Boolean,
      default () {
        return false
      }
    },
    title: String
  },
  methods: {
    close () {
      this.$emit('update:show', false)
    }
  }
}
</script>

<style lang="scss" scoped>
.modal-enter-active,
.modal-leave-active {
  transition: all 0.6s;
  transform: translateX(110%);
}
.modal-enter-to {
  transform: translateX(0);
}
.cus-modal {
  width: 500px;
  outline: none;
  display: flex;
  flex-direction: column;
  position: fixed;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: 2000;
  background-color: #ffffff;
  box-shadow: 3px 3px 14px 0 #888888;
  .header {
    flex: 0 0 80px;
    display: flex;
    align-items: center;
    padding: 0 20px;
    font-size: 20px;
    color: #333333;
    font-weight: 600;
    border-bottom: 1px solid #e3e3e3;
  }
  .content {
    padding: 10px 20px;
    margin-right: 4px;
    flex: 1;
    overflow-y: auto;
    &::-webkit-scrollbar {
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      background-color: #a3a3a3;
    }
  }
  .footer {
    flex: 0 0 80px;
    padding: 0 20px;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    border-top: 1px solid #e3e3e3;
  }
  .close {
    text-align: center;
    position: absolute;
    top: 2px;
    left: -44px;
    cursor: pointer;
    box-shadow: -3px 2px 8px 0 #b1b1b1;
    .el-icon-close {
      width: 40px;
      height: 40px;
      font-weight: 700;
      line-height: 40px;
      font-size: 26px;
      color: #666666;
      background-color: #fff;
    }
  }
}
</style>
