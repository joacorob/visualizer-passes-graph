<template>
    <div id="app">
        <div class="container">
            <section class="py-5">
                <div class="row mt-5">
                    <div class="col-8 offset-2">
                        <h4 class="mb-4">Import:</h4>
                        <vue-csv-import v-model="csv" :map-fields="['from', 'to', 'type', 'team']"></vue-csv-import>
                        <v-radio-group>
                            <v-radio
                              v-for="n in 3"
                              :key="n"
                              :label="`Radio ${n}`"
                              :value="n"
                            ></v-radio>
                        </v-radio-group>
                        <button v-on:click="take_passes('Home')"> Home </button>
                        <button v-on:click="take_passes('Away')"> Away </button>
                        <d3-network :net-nodes="nodes" :net-links="links" :options="options" />
                        <div class="mt-2">
                            {{ passes }}
                        </div>
                        <div class="mt-2">
                            {{ csv }}
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </div>
</template>

<script>
    import D3Network from 'vue-d3-network'

    export default {
        name: "app",
        components: {
            D3Network
        },
        data() {
            return {
                csv: null,
                passes: null,
                nodes: [
                    { id: 1, name: 'my awesome node 1'},
                    { id: 2, name: 'my node 2'},
                    { id: 3, name:'orange node', _color: 'orange' },
                    { id: 4, _color: '#4466ff'},
                    { id: 5 },
                    { id: 6 },
                    { id: 7 },
                    { id: 8 },
                    { id: 9 }
                ],
                links: [],
                nodeSize:20,
                canvas:false
            };
        },
        computed:{
            options(){
              return{
                force: 3000,
                size:{ w:600, h:600},
                nodeSize: this.nodeSize,
                nodeLabels: true,
                linkLabels:true,
                canvas: this.canvas
              }
            }
        },
        methods: {
            take_passes(team){
                console.log(this.csv)
                this.passes = this.csv.filter((a) => a.type == 'PASS' && a.team == team)
                var froms = new Set(this.passes.map((a) => a.from))
                var i = 1
                var nodes = []
                froms.forEach((item) => {
                  nodes.push( { name: item, id: i } )
                  i++
                })
                this.nodes = nodes
                this.links = [
                    { sid: 1, tid: 2, _color:'red', name: 'custom name' },
                    { sid: 2, tid: 8, _color:'f0f' },
                    { sid: 3, tid: 4, _color:'rebeccapurple' },
                    { sid: 3, tid: 4, _svgAttrs:{'stroke-width':8,opacity:1}  },
                    { sid: 4, tid: 5 },
                    { sid: 5, tid: 6 },
                    { sid: 7, tid: 8 },
                    { sid: 5, tid: 8 },
                    { sid: 3, tid: 8 },
                    { sid: 7, tid: 9 }
                ]
                var links = []
                for (var i = nodes.length - 1; i >= 0; i--) {
                    var passes_from_i = this.passes.filter((a) => a.from == nodes[i].name)
                    if (passes_from_i.length > 0) {
                        for (var j = 0; j < passes_from_i.length; j++) {
                            var to_name = passes_from_i[j].to
                            var to_id   = nodes.find((a) => a.name == to_name).id
                            if ( links.filter((a) => a.sid == nodes[i].id && a.tid == to_id).length == 0 ) {
                                links.push({ sid: nodes[i].id, tid: to_id, _color:'red' })
                            }
                        }
                        console.log(links)
                    }
                }
                this.links = links
                debugger
            }
        }
    };
</script>

<style>
    #app {
        font-family: "Avenir", Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    .container {
        text-align: left;
    }

    pre code {
        background-color: #eee;
        border: 1px solid #999;
        display: block;
        padding: 20px;
    }

    #app .form {
        text-align: left;
    }
</style>
