<template>
    <div id="app">
        <div class="container">
            <section class="py-5">
                <div class="row mt-5">
                    <div class="col-8 offset-2">
                        <h4 class="mb-4">Import:</h4>
                        <vue-csv-import v-model="csv" :map-fields="['from', 'to', 'type', 'team']"></vue-csv-import>
                        <input type="radio" id="uno" value="Home" v-model="team">
                        <label for="uno">Home</label>
                        <br>
                        <input type="radio" id="Dos" value="Away" v-model="team">
                        <label for="Dos">Away</label>
                        <br>
                        <button v-on:click="take_passes()"> Graph Passes </button>
                        <d3-network :net-nodes="nodes" :net-links="links" :options="options" />
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
                    { id: 4, _color: '#4466ff', _size: 30},
                    { id: 5 },
                    { id: 6 },
                    { id: 7 },
                    { id: 8 },
                    { id: 9 }
                ],
                links: [],
                nodeSize:20,
                canvas:false,
                team: 'Home',
                colors: ['red', 'blue', 'green', 'yellow']
            };
        },
        computed:{
            options(){
              return{
                force: 3000,
                size:{ w:1200, h:1200},
                nodeSize: this.nodeSize,
                nodeLabels: true,
                linkLabels:true,
                canvas: this.canvas
              }
            }
        },
        methods: {
            take_passes(){
                var team = this.team
                this.passes = this.csv.filter((a) => a.type == 'PASS' && a.team == team)
                var froms = new Set(this.passes.map((a) => a.from))
                var i = 1
                var nodes = []
                froms.forEach((item) => {
                  nodes.push( { name: item, id: i, _size: this.csv.filter((a) => a.type == 'PASS' && a.team == team && a.from == item).length } )
                  i++
                })
                this.nodes = nodes
                var links = []
                for (var i = nodes.length - 1; i >= 0; i--) {
                    var passes_from_i = this.passes.filter((a) => a.from == nodes[i].name)
                    if (passes_from_i.length > 0) {
                        for (var j = 0; j < passes_from_i.length; j++) {
                            var to_name = passes_from_i[j].to
                            var to   = nodes.find((a) => a.name == to_name)
                            if ( to && links.filter((a) => a.sid == nodes[i].id && a.tid == to.id).length == 0 ) {
                                links.push({ sid: nodes[i].id, tid: to.id, _color: this.colors[ Math.floor(Math.random() * 11) % 4 ] })
                            }
                        }
                    }
                }
                this.links = links
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
