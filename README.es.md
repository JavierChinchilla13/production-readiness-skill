# production-readiness-skill (resumen en español)

Una lista de verificación probada en un lanzamiento real y un registro de decisiones, para llevar una web pequeña desde «funciona en mi computadora» hasta el lanzamiento público. Está empaquetada como un *skill* para agentes de programación con IA (Claude Code y herramientas compatibles).

Nació de un primer lanzamiento real (React + Express + MongoDB en Render, con Cloudflare, Resend, Cloudinary e ingreso con Google). La pila es un ejemplo; el razonamiento sirve en otras.

> **No es asesoría legal.** Todo lo relacionado con privacidad, menores, cookies, consumo o reseñas es un borrador que debe revisar una persona profesional en la jurisdicción correcta. Los datos de planes y paneles de terceros cambian: cada referencia lleva su fecha de verificación (en inglés, «Last verified»).

## Qué contiene
- Reglas de arquitectura y seguridad, y comprobaciones de arranque en producción.
- Despliegue (hosting, DNS, correo, base de datos) y los errores típicos.
- Cómo **medir la IP real** del visitante detrás de proxies antes de confiar en los límites de intentos.
- Privacidad, términos, cookies, edad y consentimiento, con marcas `[PLACEHOLDER]` (datos que solo da la persona titular) y `[PROPOSAL]` (propuestas que debe confirmar un profesional).
- SEO y detalles técnicos, con el sitio cerrado a buscadores hasta el día del lanzamiento.
- Respaldos, prueba de humo y lista de verificación del lanzamiento.
- **Decisiones que no tomamos** y cuándo sí conviene tomarlas.
- **Cuándo consultar a una abogada o abogado**, y cómo preparar una consulta corta y barata.

## Instalación
Copia la carpeta `production-readiness` a tu carpeta de skills, por ejemplo `~/.claude/skills/`, y pide a tu agente que use el skill `production-readiness` al preparar un lanzamiento.

## Buenas prácticas
1. Dile al agente tu país, tu público (¿menores? ¿pagos? ¿datos sensibles?) y tu hosting.
2. Trabaja por fases y detente en cada punto de revisión.
3. Que el agente nunca haga push, despliegue ni cambios en paneles de nube sin tu orden explícita.
4. Guarda las claves en un gestor de contraseñas; no las pegues en el chat.

El contenido completo está en inglés (README.md y carpeta `production-readiness`). Licencia MIT, sin garantía.
