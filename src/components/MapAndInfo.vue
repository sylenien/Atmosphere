<template>
  <div class="hello" style='margin: 0 20px'>
    <div class='row'>
      <div class='col-md-8' style='background-color:grey; padding:0!important'>
        <div style=" height: 600px; border: 1px solid rgba(52,58,64,1)" id="mapContainer">
          <div class='panel dl-button' style='position: absolute; bottom: 0; right: 0; z-index: 10; padding-top: 5px; text-align: left'>
            Превышение ПДК
            <div>
              <div class='legend-color' style='background-color: rgba(0,255,0,1);'></div>
              <span>< 25%</span>
            </div>
            <div>
              <div class='legend-color' style='background-color: rgba(255,255,0,1);'></div>
              <span>25 - 50%</span>
            </div>
            <div>
              <div class='legend-color' style='background-color: rgba(255,125,0,1);'></div>
              <span>50 - 75%</span>
            </div>
            <div>
              <div class='legend-color' style='background-color: rgba(255,0,0,1);'></div>
              <span>> 75%</span>
            </div>
          </div>
        </div>
        </div>
      <div class='col-md-4'>
        <div class='bubble b-top'>
          <h4>Средние значения по СПб</h4>
          <ul class='text-left'>
            <li>CO: {{(median.co*100).toFixed(1)}}% ПДК  <span v-if="median.no>1" class='alert'> <i class="fas fa-exclamation"></i> Превышение ПДК</span></li> <br>
            <li>NO: {{(median.no*100).toFixed(1)}}% ПДК <span v-if="median.no>1" class='alert'> <i class="fas fa-exclamation"></i> Превышение ПДК</span></li> <br>
            <li>NO<i><sub>2</sub></i>: {{(median.no2*100).toFixed(1)}}% ПДК <span v-if="median.no2>1" class='alert'> <i class="fas fa-exclamation"></i> Превышение ПДК</span></li> <br>
            <li>SO<i><sub>2</sub></i>: {{(median.so2*100).toFixed(1)}}% ПДК <span v-if="median.no>1" class='alert'> <i class="fas fa-exclamation"></i> Превышение ПДК</span></li> <br>
          </ul>
        </div>
        <div class="bubble b-bottom">
          <div id='notLots'><h4>Нажмите на любой район, что бы увидеть сводку данных о нём.</h4> </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'MapAndInfo',
  data() {
    return {
      "markers": {},
      "toxics": {},
      "median": {"co":0, "no":0, "no2":0, "so2":0},
      "pollution": {}
    }
  },
mounted() {
    axios.get('http://localhost:8085/api/toxic')
    .then(response => {
      this.toxics = response.data;
      let query = response.data,
          obj = {"co":0, "no": 0, "no2": 0, "so2":0};
      query.forEach((item, i, items) => {
        item.co = item.co.replace(/,/g, '.');
        item.no = item.no.replace(/,/g, '.');
        item.no2 = item.no2.replace(/,/g, '.');
        item.so2 = item.so2.replace(/,/g, '.');
        obj.co += Number(item.co);
        obj.no += Number(item.no);
        obj.no2 += Number(item.no2);
        obj.so2 += Number(item.so2);
        return obj;
      });
      this.median.co = obj.co/query.length;
      this.median.no = obj.no/query.length;
      this.median.no2 = obj.no2/query.length;
      this.median.so2 = obj.so2/query.length;


      /* Creating map */
      let platform = new H.service.Platform({
        'app_id': 'xxkQMf84Rb3cKA0fF8Du',
        'app_code': '3oHkXZJOAtM3jtOvDuGd0g' 
      });
      let defaultLayers = platform.createDefaultLayers({lg: 'rus'});
      let map = new H.Map(
        document.getElementById('mapContainer'),
        defaultLayers.normal.map, 
        {
          zoom: 10,
          center: { lat: 59.939095, lng: 30.315868},
        }
      );

      /* Adding standard event map */
      let mapEvents = new H.mapevents.MapEvents(map),
          behavior = new H.mapevents.Behavior(mapEvents);

      /* Map UI */
      let ui = H.ui.UI.createDefault(map, defaultLayers, 'ru-RU'),
          mapSettings = ui.getControl('mapsettings'),
          zoom = ui.getControl('zoom'),
          scalebar = ui.getControl('scalebar');
      mapSettings.setAlignment('top-right');
      zoom.setAlignment('middle-right');
      scalebar.setAlignment('top-right');
      
      const types = ['All', 'NO', 'NO2', 'CO', 'SO2'];
      
      /* Handling query info*/
      let pollution = query,
          polyStyle = [];
      this.pollution = pollution;
      
      for(let i = 0; i < pollution.length; i++){
        polyStyle.push({
          "no": parseFloat(pollution[i].no.replace(/,/, '.'))*100,
          "no2": parseFloat(pollution[i].no2.replace(/,/, '.'))*100,
          "co": parseFloat(pollution[i].co.replace(/,/, '.'))*100,
          "so2": parseFloat(pollution[i].so2.replace(/,/, '.'))*100,
          "summary": (parseFloat(pollution[i].no2.replace(/,/, '.'))*100+parseFloat(pollution[i].no.replace(/,/, '.'))*100+parseFloat(pollution[i].so2.replace(/,/, '.'))*100+parseFloat(pollution[i].co.replace(/,/, '.'))*100)/4
        });            
      }

      let colorsPolyAll = polyStyle.map(item => item.summary > 25 ? item.summary > 50 ? item.summary > 75 ? "rgba(255,0,0,0.5)":"rgba(255,125,0,0.5)":"rgba(255,255,0,0.5)":"rgba(0,255,0,0.5)");
      let colorsPolyNO = polyStyle.map(item => item.no > 25 ? item.no > 50 ? item.no > 75 ? "rgba(255,0,0,0.5)":"rgba(255,125,0,0.5)":"rgba(255,255,0,0.5)":"rgba(0,255,0,0.5)");
      let colorsPolyNO2 = polyStyle.map(item => item.no2 > 25 ? item.no2 > 50 ? item.no2 > 75 ? "rgba(255,0,0,0.5)":"rgba(255,125,0,0.5)":"rgba(255,255,0,0.5)":"rgba(0,255,0,0.5)");
      let colorsPolyCO = polyStyle.map(item => item.co > 25 ? item.co > 50 ? item.co > 75 ? "rgba(255,0,0,0.5)":"rgba(255,125,0,0.5)":"rgba(255,255,0,0.5)":"rgba(0,255,0,0.5)");
      let colorsPolySO2 = polyStyle.map(item => item.so2 > 25 ? item.so2 > 50 ? item.so2 > 75 ? "rgba(255,0,0,0.5)":"rgba(255,125,0,0.5)":"rgba(255,255,0,0.5)":"rgba(0,255,0,0.5)");
      let groupAll = new H.map.Group(),
          groupNO2 = new H.map.Group(),
          groupSO2 = new H.map.Group(),
          groupNO = new H.map.Group(),
          groupCO = new H.map.Group();
      
      /* Getting district boarders */
      const jsx = 'http://localhost:8085/api/geojson';

      axios.get(jsx).then((response) => {
        let geodata = response.data,
            lineString;
        
        geodata.forEach((item, i, items) => {
          lineString = new H.geo.LineString()
          item.coords.forEach((item)=>{
            lineString.pushPoint(new H.geo.Point(item[1],item[0]));
          });
        
          /* Creation of polygones */
          types.forEach((type, t, ts) => {
            let pol = new H.map.Polygon(lineString, {
              style: {
                fillColor: eval("colorsPoly"+type)[i],
                  strokeColor: 'darkgrey',
                  lineWidth: 1.5
              },
              data: i
            })
            pol.addEventListener('pointerenter', function(evt) {
              let color = evt.target.style.fillColor.slice(0, -4);
              let opacity = '0.6)';
              evt.target.setStyle({fillColor:color+opacity, strokeColor: 'darkgrey'});
            });
            pol.addEventListener('pointerleave', function(evt) {
              let color = evt.target.style.fillColor.slice(0, -4);
              let opacity = '0.5)';
              evt.target.setStyle({fillColor:color+opacity, strokeColor: 'darkgrey'});
            })
            pol.addEventListener('tap', function(evt) {
              let info = pollution[evt.target.getData()]
              document.getElementById('notLots').innerHTML = `
              <h4>${info.title}</h4>
              <ul class='text-left'>
                <p>Содержание вредных веществ, % от ПДК.</p>
                <li>CO: ${info.co > 1 ? Math.round(info.co*100)+"<span class='alert'> <i class='fas fa-exclamation'></i> Превышение ПДК</span>":Math.round(info.co*100)} </li> <br>
                <li>NO: ${info.no > 1 ? Math.round(info.no*100)+"<span class='alert'> <i class='fas fa-exclamation'></i> Превышение ПДК</span>":Math.round(info.no*100)}    </li> <br>
                <li>NO<i><sub>2</sub></i>: ${info.no2 > 1 ? Math.round(info.no2*100)+"<span class='alert'> <i class='fas fa-exclamation'></i> Превышение ПДК</span>":Math.round(info.no2*100)} </li> <br>
                <li>SO<i><sub>2</sub></i>: ${info.so2 > 1 ? Math.round(info.so2*100)+"<span class='alert'> <i class='fas fa-exclamation'></i> Превышение ПДК</span>":Math.round(info.so2*100)} </li> <br>
              </ul>
              `
            });
            eval('group' + type).addObject(pol);
          });
        });
        
        /* Adding created groups into map */
        for (let i in types) {
          let gr = eval('group' + types[i]);
          gr.setVisibility(false);
          map.addObject(gr);
        }

        /* Creation of buttons */
        // TODO: refactor
        const buttonNo = new Button('NO', 'NO');
        buttonNo.setAlignment('top-left');
        ui.addControl('buttonNo',buttonNo);
        document.querySelector('.NO').addEventListener('click', ()=>{
          document.querySelector('.NO').classList.add('dl-button-active');
          document.querySelector('.NO2').classList.remove('dl-button-active');
          document.querySelector('.CO').classList.remove('dl-button-active');
          document.querySelector('.SO2').classList.remove('dl-button-active');            
          document.querySelector('.All').classList.remove('dl-button-active');            
          groupNO.setVisibility(true);
          groupCO.setVisibility(false);
          groupAll.setVisibility(false);
          groupNO2.setVisibility(false);
          groupSO2.setVisibility(false);
        })
        
        const buttonSo2 = new Button('SO2', 'SO2');
        buttonSo2.setAlignment('top-left');
        ui.addControl('buttonSo2',buttonSo2);
        document.querySelector('.SO2').addEventListener('click', ()=>{
          document.querySelector('.SO2').classList.add('dl-button-active');
          document.querySelector('.NO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.CO').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.All').classList.remove('dl-button-active'); 
          groupSO2.setVisibility(true);
          groupCO.setVisibility(false);
          groupNO.setVisibility(false);
          groupNO2.setVisibility(false);
          groupAll.setVisibility(false);
        })
        
        const buttonNo2 = new Button('NO2', 'NO2');
        buttonNo2.setAlignment('top-left');
        ui.addControl('buttonNo2',buttonNo2);
        document.querySelector('.NO2').addEventListener('click', ()=>{
          document.querySelector('.NO2').classList.add('dl-button-active');
          document.querySelector('.SO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.CO').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.All').classList.remove('dl-button-active'); 
          groupNO2.setVisibility(true);
          groupCO.setVisibility(false);
          groupNO.setVisibility(false);
          groupAll.setVisibility(false);
          groupSO2.setVisibility(false);
        })
        
        const buttonCo = new Button('CO', 'CO');
        buttonCo.setAlignment('top-left');
        ui.addControl('buttonCo',buttonCo);
        document.querySelector('.CO').addEventListener('click', ()=>{
          document.querySelector('.CO').classList.add('dl-button-active');
          document.querySelector('.NO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.SO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.All').classList.remove('dl-button-active'); 
          groupCO.setVisibility(true);
          groupAll.setVisibility(false);
          groupNO.setVisibility(false);
          groupNO2.setVisibility(false);
          groupSO2.setVisibility(false);
        })
        
        const buttonAll = new Button('All', 'All');
        buttonCo.setAlignment('top-left');
        ui.addControl('buttonAll',buttonAll);
        document.querySelector('.All').addEventListener('click', ()=>{
          document.querySelector('.All').classList.add('dl-button-active');
          document.querySelector('.NO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.SO2').classList.remove('dl-button-active');
          document.querySelector('.NO').classList.remove('dl-button-active');
          document.querySelector('.CO').classList.remove('dl-button-active'); 
          groupAll.setVisibility(true);
          groupCO.setVisibility(false);
          groupNO.setVisibility(false);
          groupNO2.setVisibility(false);
          groupSO2.setVisibility(false);
        })
        
        /* Default view */
        document.querySelector('.All').classList.add('dl-button-active');
        groupAll.setVisibility(true);
      });
        
      /* Adding markers*/
      axios.get('http://localhost:8085/api/markers').then(response => {
        this.markers = response.data.data;           
        let coords = {}, marker = [], geomarker = [], bubble = [];
        for(let i = 1; i < this.markers.length; i++) {
          coords = {lat: this.markers[i].loc.lt, lng: this.markers[i].loc.ln},
          marker[i] = new H.map.Marker(coords)
          marker[i].text = this.markers[i].title;
          marker[i].bubpos = marker[i].getPosition();
          marker[i].bubble = new H.ui.InfoBubble(marker[i].getPosition(), {content:marker[i].text});
          marker[i].bubble.close();
          
          ui.addBubble(marker[i].bubble)
          marker[i].addEventListener('tap', function(evt) {
            // retrieve maximum zoom level
            let maxZoom = 14;//target.getData().maxZoom;
            // calculate best camera data to fit object's bounds
            let cameraData = map.getCameraDataForBounds(evt.target.getGeometry().getBounds());
            map.setZoom(Math.min(cameraData.zoom, maxZoom), true);
            map.setCenter(cameraData.position, true); 
            
          });
            map.addObject(marker[i]);
        }

        /* Showing popup message on marker hover */
        let hoveredObject;
        let infoBubble = new H.ui.InfoBubble({lat: 0, lng: 0}, {});
        infoBubble.addClass('gdp-info-bubble');
        infoBubble.close();
        ui.addBubble(infoBubble);

        map.addEventListener('pointermove', (e) => {
          if(e.target != map && !e.target.getBounds){
            if (hoveredObject && hoveredObject !== e.target && e.target == map) {
                infoBubble.close();
            }

            hoveredObject = e.target;
                let row = hoveredObject.text;
                if (row) {
                    infoBubble.setContent(row)
                    infoBubble.setPosition(hoveredObject.bubpos);
                    infoBubble.open();
                }
          } else {
            infoBubble.close();
          }
        });
      })
      .catch(error => console.log(error))
    })
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.H_ib_body{
    font-size: 0.95rem;
    padding: 5px 15px 5px;
    border-radius: 10px;
}
.gdp-info-bubble .H_ib_body{
  /* margin-right: -4em; */
  margin-bottom: 18px;
  right: 30px
}
.gdp-info-bubble .H_ib_tail{
  margin-bottom: 18px;
}
.H_ib_tail {
      display: none!important;
}
.H_ib_close{
  display:none!important;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.H_overlay{
  padding-left: 0px;
  padding-right: 0px;
  text-transform: lowercase;
}
.H_overlay div{
  text-transform: capitalize;
}

.H_rdo_buttons .H_active, .H_rdo li.H_active {
    background: #4A5B65;
    border-radius: 0em;
}
.bubble{
  padding: 15px 15px 15px;
  border-radius: 15px;
  border: 1px solid #4A5B65;
  background-color: #f7f7f7;
  margin-bottom: 30px;
  transition: all .2s ease-in-out;
  box-shadow: 0 0 20px rgba(0,0,0,.4);
}
@media (max-width: 900px){
  .b-top{
    margin-top: 30px;
  }
}
.bubble:hover{
  background-color: #f0f0f0;
  box-shadow: 0 0 20px rgba(0,0,0,.6);
}
.alert{
  color: red
}
.dl-button-active {
  box-shadow: 0px 0px 5px 3px #65cfb7;
}
.legend-color {
  width: 10px;
  height: 10px;
  display: inline-block;
  margin-right: 5px;
  margin-left: 30px;
}
</style>