# MongoDB ReplicaSet TP

Este proyecto configura un ReplicaSet de MongoDB para un trabajo práctico, incluyendo nodos `Primary`, `Secondary`, un `Arbiter`, y un nodo `Secondary` con retraso de replicación.

## Estructura del ReplicaSet

- **Primary**: Nodo principal donde se realizan las operaciones de escritura.
- **Secondary**: Nodo secundario que replica los datos del Primary.
- **Arbiter**: Nodo sin datos que ayuda en elecciones de Primary.
- **Delayed Secondary**: Nodo secundario con retraso de 120 segundos en la replicación (`secondaryDelaySecs`).

## Pasos para Configurar

### 1. Iniciar las Instancias de MongoDB

Ejecuta en distintas terminales:
```bash
mongod --replSet rs --dbpath ~/data/db/rs0 --port 27017 --fork --logpath ~/data/db/rs0/mongod.log
mongod --replSet rs --dbpath ~/data/db/rs1 --port 27018 --fork --logpath ~/data/db/rs1/mongod.log
mongod --replSet rs --dbpath ~/data/db/rs2 --port 27019 --fork --logpath ~/data/db/rs2/mongod.log
```

### 2. Configurar el ReplicaSet
Conéctate al Primary y ejecuta:
```bash
cfg = {
  _id: "rs",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019", arbiterOnly: true }
  ]
};
rs.initiate(cfg);
```
### 3. Añadir un Nodo con Retraso de Replicación
Inicia el nuevo nodo:
```bash
mongod --replSet rs --dbpath ~/data/db/rs3 --port 27020 --fork --logpath ~/data/db/rs3/mongod.log
```
Luego, agrégalo al ReplicaSet:
```bash
let cfg = rs.conf();
cfg.members.push({
  _id: 3,
  host: "localhost:27020",
  priority: 0,
  secondaryDelaySecs: 120
});
rs.reconfig(cfg);
```
### 4. Verificación
Conéctate a cada nodo y usa rs.status() para verificar el estado de los nodos. Inserta documentos en el Primary y confirma la replicación en los nodos secundarios.

### Notas
Configura setReadPref("primaryPreferred") para permitir lecturas en los Secondary.
Usa db.facturas.count() para verificar la replicación de datos entre nodos.
