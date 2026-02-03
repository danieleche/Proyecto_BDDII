## 🛠️ Instalación y Despliegue

Sigue estos pasos para levantar el entorno:

1. **Clonar el repositorio:**
   ```bash
   git clone [https://github.com/danieleche/Proyecto_BDDII.git](https://github.com/danieleche/Proyecto_BDDII.git)
   cd Proyecto_BDDII
   ```
2. **Levantar CLuster**
   ```bash
   docker-compose down -v
   docker-compose up -d
   ```
## ⚙️ Configuración del Clúster (Asociar Workers)
1. **Acceder al nodo Coordinador**

   ```bash
   docker exec -it citus_coordinator_bd psql -U postgres -d coorddb
   ```
2. **Registrar nodos en la consola SQL**
   ```bash
   SELECT master_add_node('citus_worker_1_bd', 5432);
   SELECT master_add_node('citus_worker_2_bd', 5432);
   SELECT master_add_node('citus_worker_3_bd', 5432);
   ```
4. **Verificación final**
   ```bash
   SELECT * FROM master_get_active_worker_nodes();
   ```
