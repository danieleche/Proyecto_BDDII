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

## 💫 Creación de Shards y Distribución de Datos
1. **Fragmentación Horizontal Primaria (Sede)**
   ```bash
   CREATE TABLE Sede (
   id_sede INT PRIMARY KEY,
   nombre VARCHAR(100) NOT NULL,
   direccion VARCHAR(255),
   region VARCHAR(50) NOT NULL,
   CHECK (region IN ('Capital', 'Occidente', 'Oriente'))
   );

   SELECT create_distributed_table('Sede', 'id_sede');
   ```
2. **Fragmentación Horizontal Derivada (Sección)**
   ```bash
   CREATE TABLE Seccion (
   id_seccion INT,
   id_asignatura INT, 
   id_sede INT,
   codigo_seccion VARCHAR(10),
   PRIMARY KEY (id_seccion, id_sede),                                   
   FOREIGN KEY (id_sede) REFERENCES Sede(id_sede)
   );

   SELECT create_distributed_table('Seccion', 'id_sede');
   ```
3. **Fragmentación Horizontal Derivada (Evaluación)**
   ```bash
   CREATE TABLE Evaluacion (
   id_evaluacion INT,
   id_seccion INT,
   id_sede INT,
   nombre VARCHAR(100) NOT NULL,
   tipo_evaluacion VARCHAR(50),
   fecha DATE,
   porcentaje DECIMAL(5,2),
   PRIMARY KEY (id_evaluacion, id_sede),
   FOREIGN KEY (id_seccion, id_sede) REFERENCES Seccion(id_seccion, id_sede)
   );

   SELECT create_distributed_table('Evaluacion', 'id_sede');
   ```
4. **Fragmentación Vertical (Entrega)**
   ```bash
   CREATE TABLE Entrega_Seguimiento (     
   id_entrega INT,
   id_evaluacion INT,
   id_estudiante INT,
   id_sede INT,
   fecha_entrega DATE,
   estado_entrega VARCHAR(30) DEFAULT 'No entregado',
   PRIMARY KEY (id_entrega, id_sede),
   CHECK (estado_entrega IN ('Entregado', 'No entregado'))
   );

   CREATE TABLE Entrega_Contenido (
   id_entrega INT,
   id_sede INT,
   archivo_url TEXT, -- Atributo separado verticalmente
   PRIMARY KEY (id_entrega, id_sede),
   FOREIGN KEY (id_entrega, id_sede) REFERENCES Entrega_Seguimiento(id_entrega, id_sede)
   );

   SELECT create_distributed_table('Entrega_Seguimiento', 'id_sede');
   SELECT create_distributed_table('Entrega_Contenido', 'id_sede');
   ```
