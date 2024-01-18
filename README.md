# Creación de Reconocimiento Facial en Tiempo Real con JavaScript
Hacemos una aplicación de reconocimiento facial usando la librería [face-api]

#Este Proyecto es usado en proyectos reales de: 
1. Salou TV  web: https://www.saloutv.com
2. Salou Fitness TV   web: https://www.saloufitness.tv

#gracias al equipo de Vilapubli.com


1. Sigue este tutorial para crear una aplicación de Reconocimiento Facial a Tiempo Real a través de la webcam utilizando JavaScript utilizando la librería face-api de javascript
Librería face-api.js

2. Primero de todo nos descargaremos lo que necesitamos de la librería face-api.js. Necesitaremos la carpeta weights y el archivo face-api.min.js.

3. Los colocaremos dentro de nuestro proyecto y los añadiremos al archivo, en este ejemplo, index.html para poderlos usar.

#  Iniciamos la webcam
1. Para iniciar la webcam tenemos que crear un elemento <video></video> dentro de nuestro proyecto para reproducir el vídeo que recibamos de la webcam. El elemento vídeo lo definiremos como constante para usarlo a lo largo del código:

const elVideo = document.getElementById('video')

2. Para acceder a la webcam usaremos la función de navegador Navigator.getUserMedia. El cual nos devuelve el vídeo recibido de la webcam. Lo colocaremos dentro de la función cargarCamera()

navigator.getMedia = (navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia)

const cargarCamera = () => {
    navigator.getMedia(
        {
            video: true,
            audio: false
        },
        stream => elVideo.srcObject = stream,
        console.error
    )
}


# Cargar los modelos de face-api.js

1. Antes de empezar a reproducir el contenido de la webcam y empezar con el reconocimiento facial necesitamos incluir todos los modelos que hemos copiado de la librería face-api.js. Para ello usaremos la función Promise.all() para cargarlos de forma asíncrona y cuando termine, iniciar la webcam a través de la función antes creada cargarCamera()

Promise.all([
    faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
    faceapi.nets.ageGenderNet.loadFromUri('/models'),
    faceapi.nets.faceExpressionNet.loadFromUri('/models'),
    faceapi.nets.faceLandmark68Net.loadFromUri('/models'),
    faceapi.nets.faceLandmark68TinyNet.loadFromUri('/models'),
    faceapi.nets.faceRecognitionNet.loadFromUri('/models'),
    faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
    faceapi.nets.tinyFaceDetector.loadFromUri('/models'),
]).then(cargarCamera)

# Leer el contenido de la webcam
1. Para leer el contenido de la webcam, crearemos un addEventListener en el objeto video que hemos creado al principio (elVideo) para añadir allí el reconocimiento facial una vez empiece a reproducirse:

elVideo.addEventListener('play', () => {
    ...
}
# Añadir Canvas
1. Face-api.js tiene una función que se llama createCanvasFromMedia que nos permite crear un canvas con el contenido del reconocimiento facial. Éste nos crea un canvas el cual podemos añadir al documento sin problemas:

const canvas = faceapi.createCanvasFromMedia(elVideo)
document.body.append(canvas)

2. Pero a este canvas hay que redimensionarlo al tamaño del vídeo para que encaje correctamente con el contenido de la webcam:

const displaySize = {
    width: elVideo.width,
    height: elVideo.height
}
faceapi.matchDimensions(canvas, displaySize)

# Empezar el reconocimiento facial

1. Como en este caso estamos usando un vídeo, tenemos que actualizar constantemente el contenido del reconocimiento facial. Para tal cosa, usaremos el setInterval() para que vaya actualizándolo contínuamente:

setInterval(() => {
    ...
})

2. Para empezar con las detecciones de cara usaremos la función de face-api.js detectAllFaces() y le añadiremos todos los modelos que hemos cargado al principio:

const detections = await faceapi.detectAllFaces(elVideo)
    .withFaceLandmarks()
    .withFaceExpressions()
    .withAgeAndGender()
    .withFaceDescriptors()

3.  Como esta función es asíncrona, tendremos que añadir la acción de async en la función del setInterval. Así que lo modificaremos a:

setInterval(async () => {
    const detections = await faceapi.detectAllFaces(elVideo)
        .withFaceLandmarks()
        .withFaceExpressions()
        .withAgeAndGender()
        .withFaceDescriptors()
    ...
}


# Lo siguiente que tenemos que hacer es colocar cada una de las detecciones al sitio correcto. Usaremos la función resizeResults de face-api.js para redimensionar el resultado al tamaño correcto del contenedor (elVideo)

const resizedDetections = faceapi.resizeResults(detections, displaySize)

# Aun no hemos empezado a dibujar el resultado. Sólo estamos cogiendo los datos y aplicando el reconocimiento facial. Pero el usuario no lo puede ver.

1. Pero como estamos en un setInterval lo que pasará es que cada vez que se ejecute el contenido de éste es que dibujaremos el reconocimiento. Pero no lo borraremos. Así que antes de empezar a dibujar el siguiente elemento lo que tendremos que hacer es limpiar el canvas actual. Desde el punto 0, 0 a su ancho y alto total:

canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height)
Dibujar el Resultado del Reconocimiento Facial
# Una vez tengamos el contenido creado, ya podremos empezar a dibujarlo y representarlo encima del vídeo de la webcam con los tamaños que hemos leído anteriormente. Para ello usaremos la función faceapi.draw.

faceapi.draw.drawDetections(canvas, resizedDetections)
faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
faceapi.draw.drawFaceExpressions(canvas, resizedDetections)
Añadir Género y Edad

#  Hemos cargado el modelo de AgeAndGender. Pero éste no tiene función propia para dibujar el contenido. Pero en cada una de las detecciones tenemos su información. Así que podemos acceder a ello a través de un forEach:

resizedDetections.forEach(detection => {
    ...
}
# Corremos todos los resizedDetections y podemos acceder al contenido de la caja dibujada (detection.detection.box). Y añadir el label que querramos usando la función DrawBox de face-api.js.

# La caja vamos a definirla así:

const box = detection.detection.box
# Y el género y edad van a ser las propiedades detection.age y detection.gender. Así que a través de DrawBox vamos a añadir el label personalizado:

new faceapi.draw.DrawBox(box, {
    label: Math.round(detection.age) + ' años ' + detection.gender
}).draw(canvas)
Resultado Final

# Si juntamos todo el código de event listener del vídeo tendríamos el siguiente código:

elVideo.addEventListener('play', async () => {
    // creamos el canvas con los elementos de la face api
    const canvas = faceapi.createCanvasFromMedia(elVideo)
    // lo añadimos al body
    document.body.append(canvas)

    // tamaño del canvas
    const displaySize = {
        width: elVideo.width,
        height: elVideo.height
    }
    faceapi.matchDimensions(canvas, displaySize)

    setInterval(async () => {
        // hacer las detecciones de cara
        const detections = await faceapi.detectAllFaces(elVideo)
            .withFaceLandmarks()
            .withFaceExpressions()
            .withAgeAndGender()
            .withFaceDescriptors()

        // ponerlas en su sitio
        const resizedDetections = faceapi.resizeResults(detections, displaySize)

        // limpiar el canvas
        canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height)

        // dibujar las líneas
        faceapi.draw.drawDetections(canvas, resizedDetections)
        faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
        faceapi.draw.drawFaceExpressions(canvas, resizedDetections)

        resizedDetections.forEach(detection => {
            const box = detection.detection.box
            new faceapi.draw.DrawBox(box, {
                label: Math.round(detection.age) + ' años ' + detection.gender
            }).draw(canvas)
        })
    })
})

# Todo este código puedes descargarlo a través de este repositorio de github.
