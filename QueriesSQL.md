# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    c.cedula,
    COUNT(cu.num_cuenta) AS cantidad_cuentas,
    SUM(cu.saldo) AS saldo_total
FROM 
    Cliente c
JOIN 
    Cuenta cu ON c.id_cliente = cu.id_cliente
GROUP BY 
    c.id_cliente, c.nombre, c.cedula
HAVING 
    COUNT(cu.num_cuenta) > 1
ORDER BY 
    saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto ELSE 0 END) AS total_depositos,
    SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto ELSE 0 END) AS total_retiros,
    (SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto ELSE 0 END) - 
     SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto ELSE 0 END)) AS diferencia
FROM 
    Cliente c
JOIN 
    Cuenta cu ON c.id_cliente = cu.id_cliente
JOIN 
    Transaccion t ON cu.num_cuenta = t.num_cuenta
GROUP BY 
    c.id_cliente, c.nombre
ORDER BY 
    diferencia DESC;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT 
    c.num_cuenta,
    cl.nombre AS cliente,
    c.tipo_cuenta,
    c.saldo
FROM 
    Cuenta c
JOIN 
    Cliente cl ON c.id_cliente = cl.id_cliente
LEFT JOIN 
    Tarjeta t ON c.num_cuenta = t.num_cuenta
WHERE 
    t.id_tarjeta IS NULL
ORDER BY 
    c.saldo DESC;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    c.tipo_cuenta,
    COUNT(DISTINCT c.num_cuenta) AS cuentas_activas,
    AVG(c.saldo) AS saldo_promedio,
    SUM(c.saldo) AS saldo_total
FROM 
    Cuenta c
WHERE 
    EXISTS (
        SELECT 1 
        FROM Transaccion t 
        WHERE t.num_cuenta = c.num_cuenta 
        AND t.fecha >= CURRENT_DATE - INTERVAL '30 days'
    )
GROUP BY 
    c.tipo_cuenta
ORDER BY 
    saldo_promedio DESC;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT
    c.id_cliente,
    c.nombre,
    c.cedula
FROM 
    Cliente c
JOIN 
    Cuenta cu ON c.id_cliente = cu.id_cliente
JOIN 
    Transaccion t ON cu.num_cuenta = t.num_cuenta
JOIN 
    Transferencia tf ON t.id_transaccion = tf.id_transaccion
WHERE 
    NOT EXISTS (
        SELECT 1 
        FROM Transaccion tr
        JOIN Retiro r ON tr.id_transaccion = r.id_transaccion
        WHERE tr.num_cuenta = cu.num_cuenta
        AND r.canal = 'cajero'
    )
ORDER BY 
    c.nombre;
```
