<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Televisión CR Mundial ULTRA PRO</title>

<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
<script src="https://cdn.jsdelivr.net/npm/dashjs@latest/dist/dash.all.min.js"></script>

<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body { background:black; color:white; font-family:Arial; text-align:center; }
h1 { color:red; }
textarea { width:90%; height:110px; margin:10px; }
button { padding:10px; margin:5px; background:red; color:white; border:none; border-radius:5px; }
.canal { background:#111; margin:5px; padding:10px; cursor:pointer; border-radius:5px; }
#videoPlayer { width:95%; height:230px; margin-top:10px; border:2px solid white; }
#audioPlayer { width:95%; margin-top:10px; }
#mensaje { color:yellow; margin-top:10px; }
</style>
</head>

<body>

<h1>📺 IPTV ULTRA PRO (Fullscreen REAL)</h1>

<textarea id="textoM3U" placeholder="Pega tu lista IPTV..."></textarea><br>

<button onclick="cargarLista()">Cargar lista</button>
<button onclick="limpiar()">Limpiar</button>

<input type="file" id="fileInput" accept=".m3u,.txt"><br>

<div id="lista"></div>

<!-- VIDEO -->
<video id="videoPlayer" controls playsinline webkit-playsinline x5-playsinline></video>

<br>

<!-- BOTÓN QUE SÍ FUNCIONA -->
<button onclick="fullScreen()">📺 Pantalla Completa REAL</button>

<audio id="audioPlayer" controls style="display:none"></audio>

<div id="mensaje">Esperando acción...</div>

<script>
let canales=[];
let hls, dash;

function cargarLista(){
 let texto=document.getElementById("textoM3U").value;
 procesar(texto);
}

function procesar(txt){
 canales=[];
 let l=txt.split(/\r?\n/);
 for(let i=0;i<l.length;i++){
  if(l[i].includes("#EXTINF")){
   let nombre=l[i].split(",").pop();
   let url="";
   for(let j=i+1;j<l.length;j++){
    if(l[j] && !l[j].startsWith("#")){ url=l[j].trim(); break; }
   }
   if(nombre && url) canales.push({nombre,url});
  }
 }
 pintar();
}

function pintar(){
 let cont=document.getElementById("lista"); cont.innerHTML="";
 canales.forEach((c,i)=>{
  let d=document.createElement("div");
  d.className="canal";
  d.innerText=c.nombre;
  d.onclick=()=>play(i);
  cont.appendChild(d);
 });
}

function play(i){
 let url=canales[i].url;
 let v=document.getElementById("videoPlayer");
 let a=document.getElementById("audioPlayer");
 let msg=document.getElementById("mensaje");

 msg.innerText="Cargando...";

 if(hls) hls.destroy();
 if(dash) dash.reset();

 v.style.display="block";
 a.style.display="none";

 // HLS
 if(url.includes(".m3u8")){
  if(Hls.isSupported()){
   hls=new Hls();
   hls.loadSource(url);
   hls.attachMedia(v);
   v.play();
   msg.innerText="HLS Player";
   return;
  }
 }

 // DASH
 if(url.includes(".mpd")){
  dash=dashjs.MediaPlayer().create();
  dash.initialize(v,url,true);
  msg.innerText="DASH Player";
  return;
 }

 // AUDIO
 if(url.match(/\.(mp3|aac|m4a)$/i)){
  v.style.display="none";
  a.style.display="block";
  a.src=url;
  a.play();
  msg.innerText="Audio Player";
  return;
 }

 // VIDEO NORMAL
 v.src=url;
 v.play().then(()=>{
  msg.innerText="HTML5 Player";
 }).catch(()=>{
  fallback(url);
 });
}

// FALLBACK
function fallback(url){
 let msg=document.getElementById("mensaje");
 let v=document.getElementById("videoPlayer");

 msg.innerText="Intentando alternativa...";

 let proxy="https://cors.isomorphic-git.org/"+url;

 v.src=proxy;
 v.play().catch(()=>{
  msg.innerHTML="❌ Bloqueado<br>👉 Abriendo externo...";
  window.open(url,'_blank');
 });
}

// 🔥 FULLSCREEN QUE SÍ FUNCIONA
function fullScreen(){
 let v = document.getElementById("videoPlayer");
 let url = v.src;

 // intentar interno
 if(v.requestFullscreen){
  v.requestFullscreen().catch(()=>abrirExterno(url));
 }else if(v.webkitRequestFullscreen){
  v.webkitRequestFullscreen();
 }else{
  abrirExterno(url);
 }

 // si Manus bloquea → abrir externo
 setTimeout(()=>{
  if(!document.fullscreenElement){
   abrirExterno(url);
  }
 },1000);
}

// 🔥 EXTERNO (SIEMPRE FUNCIONA)
function abrirExterno(url){
 if(url){
  window.open(url, "_blank");
 }else{
  alert("Primero selecciona un canal");
 }
}

function limpiar(){
 document.getElementById("textoM3U").value="";
 document.getElementById("lista").innerHTML="";
 document.getElementById("videoPlayer").src="";
 document.getElementById("audioPlayer").src="";
 canales=[];
}

// ARCHIVO
document.getElementById("fileInput").addEventListener("change",e=>{
 let f=e.target.files[0];
 let r=new FileReader();
 r.onload=x=>{
  document.getElementById("textoM3U").value=x.target.result;
  procesar(x.target.result);
 };
 r.readAsText(f);
});
</script>

</body>
</html>
