- 👋 Hi, I’m @soledad223
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
soledad223/soledad223 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import tkinter as tk
import random
from PIL import Image, ImageTk
import pygame
import math

# Inicializar pygame para reproducir sonidos
pygame.mixer.init()

# Crear la ventana principal
ventana = tk.Tk()
ventana.title("Juego de Galaxy")

# Cargar la imagen
try:
    imagen = Image.open("Galaxy Wallpaper 4K 48 images.jpg")
    imagen = imagen.resize((600, 600))
    
    imagen_tk = ImageTk.PhotoImage(imagen)
except Exception as e:
    print("Error al cargar la imagen:", str(e))

# Crear el canvas (lienzo) para el juego
canvas = tk.Canvas(ventana, width=500, height=500)
canvas.pack()

# Insertar la imagen en el fondo del canvas
canvas.imagen_tk = imagen_tk  # Mantener una referencia global a la imagen
canvas.create_image(0, 0, image=imagen_tk, anchor="nw")

# Cargar la imagen de la nave del jugador
imagen_jugador = Image.open("nave-jugador.png")
imagen_jugador = imagen_jugador.resize((50, 50))
imagen_jugador_tk = ImageTk.PhotoImage(imagen_jugador)

# Cargar la imagen de la bala del jugador
imagen_bala = Image.open("5ab114187603fc558cffc0fd.png")
imagen_bala = imagen_bala.resize((15, 15))
imagen_bala_tk = ImageTk.PhotoImage(imagen_bala)

# Cargar la imagen de la nave enemiga
imagen_enemigo = Image.open("pngwing.com (1).png")
imagen_enemigo = imagen_enemigo.resize((45, 45))
imagen_enemigo_tk = ImageTk.PhotoImage(imagen_enemigo)

# Cargar el sonido de fondo
pygame.mixer.music.load("boss-133663 (1).mp3")
pygame.mixer.music.play(-1)  # Reproducir el sonido en bucle

# Crear el jugador
jugador = canvas.create_image(225, 450, image=imagen_jugador_tk)

# Crear los enemigos
enemigos = []
for _ in range(10):
    x = random.randint(50, 450)
    y = random.randint(50, 200)
    enemigo = canvas.create_image(x, y, image=imagen_enemigo_tk)
    enemigos.append(enemigo)

# Crear los disparos
disparos = []

# Crear la puntuación
puntuacion = 0

# Crear etiqueta de puntuación
etiqueta_puntuacion = canvas.create_text(10, 10, text="Puntuación: 0", anchor="nw", fill="white", font=("Arial", 13, "bold"))

# Crear las vidas
vidas = 3

# Crear etiqueta de vidas
etiqueta_vidas = canvas.create_text(10, 30, text="Vidas: 3", anchor="nw", fill="white", font=("Arial", 13, "bold"))

# Cargar el sonido de disparo
sonido_disparo = pygame.mixer.Sound("gunshot-single.wav")

estado_juego = "jugando"  # "jugando", "ganaste" o "perdiste"

# Función para crear los disparos
def crear_disparo(event):
    sonido_disparo.play()  # Reproducir el sonido de disparo
    bbox = canvas.bbox(jugador)
    if bbox:
        x1, y1, x2, y2 = bbox
        x = (x1 + x2) // 2 - 5
        y = y1 - 30
        disparo = canvas.create_image(x, y, image=imagen_bala_tk)
        disparos.append(disparo)

# Variable si choca la nave
chocar = False

# Mover los disparos
def mover_disparos(ventana):
    disparos_a_eliminar = []
    for disparo in disparos:
        canvas.move(disparo, 0, -10)
        # Eliminar el disparo si sale del canvas
        if canvas.coords(disparo)[1] <= 0:
            disparos_a_eliminar.append(disparo)
        else:
            # Verificar colisión con los enemigos
            for enemigo in enemigos:
                if canvas.bbox(disparo) and canvas.bbox(enemigo) and canvas.bbox(disparo)[1] < canvas.bbox(enemigo)[3]:
                    dx = canvas.coords(enemigo)[0] - canvas.coords(disparo)[0]
                    dy = canvas.coords(enemigo)[1] - canvas.coords(disparo)[1]
                    distancia = math.sqrt(dx**2 + dy**2)
                    if distancia <= 35:
                        disparos_a_eliminar.append(disparo)
                        canvas.delete(enemigo)
                        enemigos.remove(enemigo)
                        aumentar_puntuacion(10)
                        break

    # Eliminar los disparos que colisionaron o salieron del canvas
    for disparo in disparos_a_eliminar:
        canvas.delete(disparo)
        disparos.remove(disparo)

    # Verificar el estado del juego
    verificar_estado_juego()
    
    # Llamar a la función mover_disparos después de 50 milisegundos
    ventana.after(50, mover_disparos, ventana)

# Llamar a la función crear_disparo al presionar la tecla de espacio
ventana.bind("<space>", crear_disparo)

# Mover el jugador con las teclas de flecha
def mover_jugador(event):
    if event.keysym == "Left":
        canvas.move(jugador, -10, 0)  # Mover hacia la izquierda
    elif event.keysym == "Right":
        canvas.move(jugador, 10, 0)  # Mover hacia la derecha

ventana.bind("<Key>", mover_jugador)

nivel = 1

# Función para aumentar el nivel
def aumentar_nivel():
    global nivel
    nivel += 1
    etiqueta_nivel.config(text="Nivel: " + str(nivel))

# Función para mantener el nivel
def mantener_nivel():
    global nivel
    nivel >= 1
    etiqueta_nivel['text'] = "Nivel: " + str(nivel)

# Función para aumentar la puntuación
def aumentar_puntuacion(puntos):
    global puntuacion
    puntuacion += puntos
    canvas.itemconfig(etiqueta_puntuacion, text="Puntuación: " + str(puntuacion))

# Función para disminuir la puntuación
def disminuir_puntuacion(puntos):
    global puntuacion
    puntuacion -= puntos
    if puntuacion < 0:
        puntuacion = 0
    canvas.itemconfig(etiqueta_puntuacion, text="Puntuación: " + str(puntuacion))

# Función para verificar el estado del juego
def verificar_estado_juego():
    global estado_juego, vidas, chocar
    if len(enemigos) == 0 and not chocar:
        if estado_juego == "jugando":
            aumentar_nivel()
            estado_juego = "ganaste"
        canvas.itemconfig(etiqueta_estado, text="¡Ganaste!", fill="green")
    elif vidas == 0:
        estado_juego = "perdiste"
        mantener_nivel()
        canvas.itemconfig(etiqueta_estado, text="¡Perdiste!", fill="red")

# Función para actualizar las vidas
def actualizar_vidas():
    canvas.itemconfig(etiqueta_vidas, text="Vidas: " + str(vidas))

# Función para pausar/reanudar el juego
def pausar_reanudar():
    global estado_juego
    if estado_juego == "jugando":
        estado_juego = "pausado"
        canvas.itemconfig(etiqueta_estado, text="Pausado", fill="orange")
    elif estado_juego == "pausado":
        estado_juego = "jugando"
        canvas.itemconfig(etiqueta_estado, text="")
        canvas.itemconfig(etiqueta_puntuacion, text="Puntuación: 0")
        canvas.itemconfig(etiqueta_vidas, text="Vidas: 3")
        mover_enemigos()
        mover_disparos(ventana)
#funcion para reiniciar juego
def reiniciar_juego():
    global estado_juego, puntuacion, vidas, enemigos, disparos, chocar
    
    # Restablecer variables
    estado_juego = "jugando"
    puntuacion = 0
    vidas = 3
    enemigos = []
    disparos = []
    chocar = False
    # Eliminar elementos del lienzo
    canvas.delete("all")
    
    # Volver a insertar la imagen en el fondo del canvas
    canvas.create_image(0, 0, image=imagen_tk, anchor="nw")
    
    # Crear al jugador
    jugador = canvas.create_image(225, 450, image=imagen_jugador_tk)
   
    def mover_jugador(event):
        if event.keysym == "Left":
            canvas.move(jugador, -10, 0)  # Mover hacia la izquierda
        elif event.keysym == "Right":
            canvas.move(jugador, 10, 0)  # Mover hacia la derecha

    ventana.bind("<Key>", mover_jugador)
    #crear los disparos
    def crear_disparo(event):
        sonido_disparo.play()  # Reproducir el sonido de disparo
        bbox = canvas.bbox(jugador)
        if bbox:
            x1, y1, x2, y2 = bbox
            x = (x1 + x2) // 2 - 5
            y = y1 - 30
            disparo = canvas.create_image(x, y, image=imagen_bala_tk)
            disparos.append(disparo)
    # Llamar a la función mover_disparos después de 50 milisegundos
    ventana.after(50, mover_disparos, ventana)

    # Llamar a la función crear_disparo al presionar la tecla de espacio
    ventana.bind("<space>", crear_disparo)
    
    # Restablecer etiquetas de puntuación y vidas
    etiqueta_puntuacion = canvas.create_text(10, 10, text="Puntuación: 0", anchor="nw", fill="white", font=("Arial", 13, "bold"))   
    etiqueta_vidas = canvas.create_text(10, 30, text="Vidas: 3", anchor="nw", fill="white", font=("Arial", 13, "bold"))
    
    # Crear nuevos enemigos
    for _ in range(10):
        x = random.randint(50, 450)
        y = random.randint(50, 200)
        enemigo = canvas.create_image(x, y, image=imagen_enemigo_tk)
        enemigos.append(enemigo)   


# Función para reiniciar la posición del jugador
def reiniciar_posicion_jugador():
    canvas.coords(jugador, 225, 450)

# Función para mover los enemigos
def mover_enemigos():
    if estado_juego == "jugando":
        for enemigo in enemigos:
            direccion = random.choice(["izquierda", "derecha"])
            canvas.move(enemigo, 0, 10)
            if direccion == "izquierda":
                canvas.move(enemigo, -10, 0)
            elif direccion == "derecha":
                canvas.move(enemigo, 10, 0)
            # Verificar colisión con el jugador
            if canvas.bbox(enemigo) and canvas.bbox(jugador) and canvas.bbox(enemigo)[3] >= canvas.bbox(jugador)[1]:
                colisionar_jugador()
                break
        ventana.after(500, mover_enemigos)


# Función para colisionar con el jugador
def colisionar_jugador():
    global vidas, chocar
    if not chocar:
        chocar = True
        vidas -= 1
        actualizar_vidas()
        reiniciar_posicion_jugador()
        canvas.itemconfig(etiqueta_estado, text="¡Chocaste!", fill="red")
        ventana.after(1000, reiniciar_chocar)

# Función para reiniciar la variable de chocar
def reiniciar_chocar():
    global chocar
    chocar = False
    canvas.itemconfig(etiqueta_estado, text="")
    verificar_estado_juego()
# Función para salir del juego
def salir_juego():
    ventana.destroy()
# Etiqueta de estado del juego
etiqueta_estado = canvas.create_text(250, 250, text="", fill="white", font=("Arial", 30))

# Etiqueta de nivel
etiqueta_nivel = tk.Label(ventana, text="Nivel: 1")
etiqueta_nivel.pack()

# Botón para pausar/reanudar el juego
boton_pausar_reanudar = tk.Button(ventana, text="Pausar/Reanudar", command=pausar_reanudar)
boton_pausar_reanudar.pack()

# crear boton para salir
boton_salir = tk.Button(ventana, text="Salir", command=salir_juego)
boton_salir.pack(side="right", padx=10)

# Botón para reiniciar el juego
boton_reiniciar = tk.Button(ventana, text="Continuar", command=reiniciar_juego)
boton_reiniciar.pack()

# Iniciar el movimiento de los enemigos
mover_enemigos()

# Iniciar el movimiento de los disparos
mover_disparos(ventana)


# Iniciar el juego
ventana.mainloop()
