<script setup lang="ts">
"use strict"

import { onMounted } from 'vue'
import Controller from './components/Controller.vue'
import ZoomCanvas from './components/ZoomCanvas.vue'

type MyCanvasElement = HTMLCanvasElement|null;
type Point = {
  x:number,
  y:number
}
let n: number = 50 //Number of turned hyperbolas


function handleDrag(dx:number,dy:number){
  console.log(handleDrag)
  translation[0] += dx*magnification
  translation[1] += -1*dy*magnification
  updateTranslation(translation[0], translation[1], gl, program)
  renderScene(gl, program)

  // console.log(dx,dy)
}

function handleMove(arg:string){
  let movementSpeed = 0.5
  if (arg == 'up'){
    translation[1] -= movementSpeed / magnification
  }
  else if (arg == 'down'){
    translation[1] += movementSpeed / magnification
  }
  else if (arg == 'left'){
    translation[0] += movementSpeed / magnification
  }
  else if (arg == 'right'){
    translation[0] -= movementSpeed / magnification
  }

  updateTranslation(translation[0], translation[1], gl, program)
  renderScene(gl, program)
}

function handleChangeMagnification(arg:string){
  if (arg =='increase'){
    magnification *= 1.1
  }else if (arg == 'decrease'){
    magnification /= 1.1
  }
  updateMagnification(magnification,gl, program)
  renderScene(gl, program)
}

//This function gets all factor pairs greater than 1 and less than n
function getFactorPairs(n:number){
  let factors = []
  for (let i = 2; i < n; i++){
    if (n % i == 0){
      let p:Point = {x:i,y:n/i}
      factors.push(p)
    }
  }
  return factors
}

let vertexShaderSource = `
    // an attribute will receive data from a buffer
    attribute vec4 position;
    uniform vec4 translation;
    uniform vec4 magnification;

    // all shaders have a main function
    void main() {
      gl_Position = (position + translation) * magnification;
    }
`

let fragmentShaderSource = `
    precision mediump float;
    uniform vec4 colour;
     
    void main() {
      gl_FragColor = colour;
    }
`


let lineVertices:number[] = [] //Float32Array
let triangleVertices:number[] = []
let curveVertices:number[] = []

let translation:number[] = [0,0] //This is not a float32. It is the conceptual model of the translation.
let gl: WebGLRenderingContext
let program: WebGLProgram
let magnification: number = 0.05

function drawLines(buffer:number[]){
  let bufferF32 = new Float32Array(buffer)
  gl.bufferData(gl.ARRAY_BUFFER, bufferF32, gl.STATIC_DRAW)

  gl.drawArrays(gl.LINES, 0, buffer.length/2);

}

function renderScene(gl:WebGLRenderingContext, program: WebGLProgram){  
  let viewportLeft = 0 
  let viewportBottom = 0 
  let canvasWidth = gl.canvas.width
  let canvasHeight = gl.canvas.height 

  gl.viewport(viewportLeft, viewportBottom, canvasWidth, canvasHeight)

  gl.clearColor(0.0, 0.0, 0.0, 1.0)
  gl.clear(gl.COLOR_BUFFER_BIT)
  gl.enable(gl.BLEND)
  gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);

  lineVertices = []
  triangleVertices = []
  curveVertices = []


  let resolution = 10 //10 points per square 1x1 in abstract coordinates
  let curve_number

  //Draw the curves
  for (curve_number = 1; curve_number <= n; curve_number++){
    let points:Point[] = []

    for (let t = 1 * resolution; t <= curve_number * resolution; t = t + 1){
      let newPoint: Point = {x:0,y:0}
      newPoint.x = t/resolution - 1
      newPoint.y = curve_number - curve_number/(curve_number - t/resolution + 1)

      points.push(newPoint)
    }

    for (let k = 0; k < points.length - 1; k++){
      addLine(points[k].x, points[k].y, points[k+1].x, points[k+1].y, curveVertices)
    }
  }


  let positionAttributeLocation = gl.getAttribLocation(program, "position")

  let positionBuffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)
  gl.enableVertexAttribArray(positionAttributeLocation)
  gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0)

  //Draw the grid
  for (let i = 0; i < n; i++){
    addLine(i,0,i,n, lineVertices)
    addLine(0,i,n,i, lineVertices)
  }

  setColour(1,1,1,1)
  drawLines(lineVertices)

  setColour(1,0,0,1)
  drawLines(curveVertices)
  



  //Draw the circles
  for (let i = 1; i <= n; i++){
    let factorPairs = getFactorPairs(i)
    for (let j = 0; j < factorPairs.length; j++){
      addCircle(i - factorPairs[j].x, i-factorPairs[j].y, 0.5, triangleVertices)
    }
  }

  // triangleVertices = [0,0,1,1,2,1,0,0,1,1,1,0]

  let triangleVerticesFloat32 = new Float32Array(triangleVertices)
  gl.bufferData(gl.ARRAY_BUFFER, triangleVerticesFloat32, gl.STATIC_DRAW)
  // gl.enableVertexAttribArray(positionAttributeLocation)
  gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0)

  setColour(1,1,0,0.7);

  gl.drawArrays(gl.TRIANGLES, 0, triangleVertices.length/2);

}

//m is the magnification
function updateMagnification(m: number,gl: WebGLRenderingContext,program: WebGLProgram){
  magnification = m
  let magnificationUniformLocation = gl.getUniformLocation(program, "magnification")

  //Reminder that component 4 is the alpha channel
  gl.uniform4f(magnificationUniformLocation,magnification,magnification,1,1)
}

//Sets the x and y offset of all points drawn
function updateTranslation(x:number,y:number, gl:WebGLRenderingContext, program: WebGLProgram){
  translation[0] = x
  translation[1] = y

  let translation4 = [...translation,0,0]
  let translationUniformLocation = gl.getUniformLocation(program, "translation")
  let translationAsFloat32Array = new Float32Array(translation4)

  gl.uniform4fv(translationUniformLocation,translationAsFloat32Array)
}

function addPoint(x:number,y:number, buffer: number[]){
  buffer.push(x * gl.canvas.height/gl.canvas.width)
  buffer.push(y)
}

function addLine(x:number, y:number, x1:number,y1:number, buffer: number[]){
  addPoint(x,y, buffer)
  addPoint(x1,y1, buffer)
}

//Draws an X through location x,y
function addX(x:number,y:number, buffer: number[]){
  addPoint(x-0.5,y-0.5, buffer)
  addPoint(x+0.5,y+0.5, buffer)

  addPoint(x+0.5,y-0.5, buffer)
  addPoint(x-0.5,y+0.5, buffer)
}

//Draws a circle at x,y with the given radius
//r is the radius
function addCircle(x:number,y:number,r:number, buffer: number[]){
  let angle:number = 0
  //x, y are the centre
  //x1,y1 are the point at angle 
  let n = 10
  let theta1, theta2
  for (let i = 0; i < n; i++){
    theta1 = 2 * Math.PI * (i /n)
    theta2 = 2 * Math.PI * (i + 1)/n

    let x1 = r*Math.cos(theta1) + x
    let y1 = r*Math.sin(theta1) + y

    let x2 = r*Math.cos(theta2) + x

    let y2 = r*Math.sin(theta2) + y

    addTriangle(x,y,x1,y1,x2,y2, buffer)
  }
}

function addTriangle(x:number,y:number,x2:number,y2:number,x3:number,y3:number, buffer: number[]){
  addPoint(x,y, buffer)
  addPoint(x2,y2, buffer)
  addPoint(x3,y3, buffer)
}

function setColour(r:number,g:number,b:number,alpha:number){
  let colour4 = [r,g,b,alpha]
  let colour4UniformLocation = gl.getUniformLocation(program, "colour")
  let colour4AsArray = new Float32Array(colour4)

  gl.uniform4fv(colour4UniformLocation,colour4AsArray)
}

onMounted(() => {
  const canvas: MyCanvasElement = document.getElementById("canvas") as MyCanvasElement
  gl = canvas?.getContext("webgl")!

  if (!canvas){
    console.log('canvas does not exist')
  }
  else{

    // let resolution = 10 //Points per 1 unit of space
    // for (i = 1; i <= n; i++){
    //   high_t = i

    //   let t
    //   let x
    //   let y
    //   for (t = low_t; t < high_t; t = t + (1 / resolution)){
    //     x = t
    //     y = i + 1 - (i/(i + 1 - x))
    //     addPoint(x,y)
    //   }
    // }

    let vertexShader = gl.createShader(gl.VERTEX_SHADER)!
    gl.shaderSource(vertexShader, vertexShaderSource)
    gl.compileShader(vertexShader)

    // if (!gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)) {
    console.log('Vertex shader log')
    console.log(gl.getShaderInfoLog(vertexShader));
    // }

    let fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)!
    gl.shaderSource(fragmentShader, fragmentShaderSource)
    gl.compileShader(fragmentShader)

    // if (!gl.getShaderParameter(fragmentShader, gl.COMPILE_STATUS)) {
    console.log('Fragment shader log')
    console.log(gl.getShaderInfoLog(fragmentShader));
    // }


    program = gl.createProgram()!
    gl.attachShader(program, vertexShader)
    gl.attachShader(program, fragmentShader)

    gl.linkProgram(program)

    console.log('ProgramInfoLog')
    console.log(gl.getProgramInfoLog(program));    
  // if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
  //   console.log(gl.getProgramInfoLog(program));
  // }
    gl.useProgram(program)

    //Magnification and translation need to be set or else shaders will use the default values
    updateMagnification(magnification,gl,program)    

    translation[0] = -(n/4)
    translation[1] = -(n / 2)
    updateTranslation(translation[0],translation[1],gl,program)


    renderScene(gl,program)




    // var primitiveType = gl.TRIANGLES;
    // var offset = 0;
    // var count = 3;
    // const [minSize, maxSize] = gl.getParameter(gl.ALIASED_LINE_WIDTH_RANGE);
    // console.log(minSize, maxSize)
  }
})

</script>

<template>
  <div>
    <Controller @move="handleMove" @changeMagnification="handleChangeMagnification"></Controller>
    <div style="display:flex; flex-direction: column;">
      <div style="flex-direction: column; flex-grow: inherit;">
        <p>
          Welcome to my number theory gallery. The canvas shows some math art that I hope you will enjoy.
        </p>
        <div style="position: relative; display:flex;">
          <ZoomCanvas v-on:drag="handleDrag"></ZoomCanvas>
        </div>
      </div>
    </div>

  </div>
</template>

<style>

#canvas{
  flex:1;
  /* left: 5vw;
  right: 5vw;
  top: 5vw;
  bottom: 5vw;
  width: 90vw;
  height: 90vw;
  position: absolute; */
}

</style>
