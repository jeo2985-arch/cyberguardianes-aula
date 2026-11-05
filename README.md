# CyberGuardianes — Plataforma educativa funcional

Aplicación web **real**, no simulación de almacenamiento. Frontend HTML/CSS/JS + backend Node.js/Express + base de datos Azure SQL. Preparada para GitHub + Render.

## Funciones implementadas

- Registro de estudiantes restringido por código de clase; contraseñas cifradas con bcrypt.
- Inicio de sesión persistente con cookie firmada, `HttpOnly`, `Secure` en producción y `SameSite=Lax`.
- Una cuenta docente inicial creada al primer arranque con variables de entorno (no existe registro público de docentes).
- 4 misiones con entregas de texto/enlaces, estados pendientes, aprobados o para corregir.
- Panel del docente con todas las evidencias y aprobación/retroalimentación.
- Cuestionario corregido **en el servidor**, intentos ilimitados y puntuación persistente.
- XP, créditos, insignias y canje cosmético calculados desde registros del servidor.
- Diseño responsive para celulares, tabletas y computadoras.
- Comprobación de salud: `/api/health`.

**Límites:** no incluye subida directa de archivos, restablecimiento automático de contraseña, importación de calificaciones, correo electrónico, múltiples clases ni informes PDF. Las evidencias pueden incluir enlaces a Google Drive con permisos apropiados. Este es un punto de partida funcional para un grupo, no un sistema escolar integral auditado.

## 1. Preparar Azure SQL

Puedes utilizar tu base `AvanzadaII` existente o crear `CyberGuardianesDB`. La aplicación crea automáticamente cuatro tablas con prefijo `cg_` al primer arranque: `cg_users`, `cg_quiz`, `cg_submissions`, `cg_rewards`. El usuario SQL debe tener permisos de creación de tablas la primera vez. Después puedes aplicar permisos mínimos para operaciones ordinarias. En Azure Networking/Firewall, permite las IP de salida **reales del servicio Render** (consultar panel de Render); no uses una lista inventada de IP. Configura cifrado y no publiques los datos de conexión.

## 2. Ejecutar localmente

1. Instala Node.js 22 o superior.
2. Copia `.env.example` a `.env`.
3. Rellena la información REAL de Azure SQL y el código secreto de clase.
4. Crea una cadena aleatoria robusta para `SESSION_SECRET` (mínimo 32 caracteres). Usa una contraseña docente **nueva** de al menos 12 caracteres.
5. Ejecuta:

```bash
npm install
npm start
```

6. Abre `http://localhost:3000`.
7. El docente entra usando `ADMIN_EMAIL` y `ADMIN_PASSWORD`. Los alumnos crean su cuenta con `CLASS_CODE`.

**Importante:** cambiar `ADMIN_PASSWORD` en Render después de crear el docente **no** cambia la contraseña ya guardada. Hay que desarrollar una función de cambio o restablecerla desde administración segura de la BD. Mantén el correo docente estable. No compartas el código de clase públicamente.

## 3. Subir a GitHub

Crea un repositorio, por ejemplo `cyberguardianes-aula`, y ejecuta en esta carpeta:

```bash
git init
git add .
git commit -m "Plataforma CyberGuardianes con Azure SQL"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/cyberguardianes-aula.git
git push -u origin main
```

**Nunca** subas `.env`, contraseñas, códigos de clase reales ni llaves. `.gitignore` excluye los archivos `.env`.

## 4. Crear Web Service en Render

1. Render → **New → Web Service** → selecciona el repositorio GitHub.
2. Language: `Node`.
3. Build Command: `npm install`.
4. Start Command: `npm start`.
5. Health Check Path: `/api/health`.
6. Environment: agrega una a una **todas** las variables de `.env.example` con valores privados reales. Pon `NODE_ENV=production`, y **no** subas el `.env`.
7. Deploy; lee los Logs hasta ver **Azure SQL conectado** y el servidor en ejecución.
8. Abre la dirección REAL asignada por Render, por ejemplo `https://<nombre-asignado>.onrender.com`. Este ejemplo no es una URL ya desplegada.

Render asigna `PORT` automáticamente; no es obligatorio configurarlo. El backend escucha en `0.0.0.0`.

## 5. Prueba con tu clase (antes de enviar URL)

1. Accede como docente y confirma que ves el panel sin estudiantes.
2. En otro navegador, registra un alumno de prueba con el código privado de clase.
3. Responde las 3 preguntas (la respuesta se guarda en Azure SQL).
4. Entrega la misión 1 con texto de más de 40 caracteres.
5. Desde el panel del profesor, solicita una corrección y escribe retroalimentación.
6. Como alumno, mejora la evidencia y reenvíala.
7. Aprueba esa evidencia desde el panel docente.
8. Como alumno, confirma que el progreso y los puntos aparecen también al cerrar sesión y volver a entrar desde otro dispositivo.
9. Comprueba que el marcador no se duplica al repetir el laboratorio.
10. Antes de utilizar datos reales de estudiantes, consulta las políticas institucionales de consentimiento, privacidad, conservación de datos y seguridad.

## Diseño didáctico

- Semana 1: Observación — identificar cinco señales de phishing.
- Semana 2: Verificación — analizar remitentes, URL y mensajes.
- Semana 3: Protección — diseñar protocolo de respuesta.
- Semana 4: Difusión — elaborar una campaña preventiva.

**La nota académica se determina mediante la rúbrica docente**; los XP y créditos son incentivos de juego y no sustituyen la evaluación.

## Seguridad y mantenimiento

Esta versión requiere conexión a Azure para iniciar; no admite modo local sin base de datos. El servicio debe permanecer conectado a la BD para registrar y consultar datos. Haz copias de seguridad con Azure SQL y utiliza HTTPS. El código de clase es compartido: alumnos que lo conozcan pueden registrarse, por lo que el docente debe compartirlo de forma controlada y renovarlo si se filtra. Para desplegar a mayor escala conviene incorporar recuperación de contraseña, gestión de consentimientos, auditoría, cuentas múltiples por institución y revisión de seguridad.


## Actualización: narrativa interactiva editable

Esta versión incluye cuatro episodios con dilemas y dos decisiones por capítulo. Desde el panel docente aparece **Editor de narrativas**: selecciona una misión, edita su transmisión, dilema, opciones, retroalimentación y cierre, y guarda. Se persiste en Azure SQL mediante `dbo.cg_stories`, creada automáticamente al iniciar la aplicación (si el usuario SQL tiene permiso `CREATE TABLE`). Los textos iniciales se siembran una sola vez; los cambios del docente no se sobrescriben al reiniciar.

Los alumnos abren cada capítulo desde el mapa de misiones o la sección Historia interactiva. Las decisiones generan comentarios formativos, sin otorgar XP ni alterar notas: solo los envíos y la aprobación docente permiten avanzar académicamente. Se conservan cuentas, progreso, cuestionarios y entregas existentes.

**Despliegue:** sustituir `server.js`, `public/index.html`, `public/app.js`, `public/style.css` y `README.md` en el repositorio GitHub, realizar commit y push. Render reconstruye la aplicación si el auto-deploy está activo. Verificar `https://TU-SERVICIO.onrender.com/api/health` y luego `/api/stories`; comprobar edición desde cuenta docente. No modificar variables secretas.
