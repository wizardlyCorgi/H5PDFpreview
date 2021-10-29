<template>
  <div class="pdf-view">
    <div id="pdf-h5"></div>
  </div>
</template>

<script>
import Pdfh5 from "pdfh5";
import "pdfh5/css/pdfh5.css";
import { getStepFile } from "@/services/api";

export default {
  name: "AgreementView",
  data() {
    return {
      pdfh5: null // pdfh5实例
    };
  },
  methods: {
    // 协议预览
    getStepFileFun(code, fileId) {
      let query = {
        Code: code,
        FileId: fileId,
        ContentDisposition: "attachment"
      };

      getStepFile(query).then(res => {
        if (res.data) {
          if (res.data.byteLength > 1000) {
            this.pdfh5 = new Pdfh5("#pdf-h5", {
              // pdfurl: src, // pdfUrl与data二选一
              data: res.data,
              lazy: true, // 懒加载
              zoomEnable: false // 禁止缩放
            });
            //监听完成事件
            this.pdfh5.on("complete", status => {
              // 加载失败
              if (status === "error") {
                this.$toast.fail(this.$t("promptMsg.getAttachmentFail"));
              }
            });
          } else {
            const result = JSON.parse(this.common.buf2str(res.data));
            if (result.code === 500) {
              let errorsMsg = "";
              if (result.errorsMsg && result.errorsMsg[0]) {
                errorsMsg = result.errorsMsg[0];
              } else {
                errorsMsg = result.message || result.errorsMsg[0];
              }
              this.$toast.fail(errorsMsg);
            }
          }
        }
      });
    },
    // 样例预览
    getExampleFile(src) {
      this.pdfh5 = new Pdfh5("#pdf-h5", {
        pdfurl: src,
        lazy: true, // 懒加载
        zoomEnable: false // 禁止缩放
      });
      //监听完成事件
      this.pdfh5.on("complete", status => {
        // 加载失败
        if (status === "error") {
          this.$toast.fail(this.$t("promptMsg.getAttachmentFail"));
        }
      });
    }
  },
  mounted() {
    const routeQuery = this.$route.query;
    if (routeQuery.src) {
      const src = routeQuery;
      this.getExampleFile(src);
    } else if (routeQuery.code) {
      const { code, fileId } = routeQuery;
      this.getStepFileFun(code, fileId);
    }
  },
  beforeDestroy() {
    // 销毁hdfh5实例
    this.pdfh5 && this.pdfh5.destroy();
  }
};
</script>

<style lang="less" scoped>
.pdf-view {
  height: 100%;
  /deep/.loadingBar {
    .glimmer {
      background-color: @theme-color;
    }
  }
}
</style>
