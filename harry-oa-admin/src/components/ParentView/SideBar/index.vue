computed: {
    ...mapGetters([
      'sidebar'
    ]),
    routes() {
      //return this.$router.options.routes
      return this.$router.options.routes.concat(global.antRouter)
    },