# Proyecto-Seguro-Autos-Base-de-Datos

DROP TABLE IF EXISTS clientes, polizas, agentes, siniestros, pagos, auditorias CASCADE;

Select *from insurance_cliente
Select *from insurance_agente
Select *from insurance_proveedor
select *from insurance_tarjeta
select *from insurance_pagos
select *from insurance_poliza

CREATE DATABASE seguro_autos;

ALTER TABLE insurance_agente ADD COLUMN contrasena VARCHAR(255);

ALTER TABLE insurance_tarjeta
ADD COLUMN id SERIAL PRIMARY KEY;

CREATE SEQUENCE seq_clientes START 1; --Generamos contadores automáticos para los id. 
CREATE SEQUENCE seq_polizas START 1; 
CREATE SEQUENCE seq_agentes START 1; 
CREATE SEQUENCE seq_siniestros START 1; 
CREATE SEQUENCE seq_pagos START 1; 
CREATE SEQUENCE seq_auditorias START 1; 

-- SECUENCIAS REGIONALES CLIENTES
CREATE SEQUENCE seq_clientes_norte START 1;
CREATE SEQUENCE seq_clientes_sur START 1;
CREATE SEQUENCE seq_clientes_este START 1;
CREATE SEQUENCE seq_clientes_oeste START 1;

-- SECUENCIAS REGIONALES PÓLIZAS
CREATE SEQUENCE seq_polizas_norte START 1;
CREATE SEQUENCE seq_polizas_sur START 1;
CREATE SEQUENCE seq_polizas_este START 1;
CREATE SEQUENCE seq_polizas_oeste START 1;

-- SECUENCIAS REGIONALES AGENTES
CREATE SEQUENCE seq_agentes_norte START 1;
CREATE SEQUENCE seq_agentes_sur START 1;
CREATE SEQUENCE seq_agentes_este START 1;
CREATE SEQUENCE seq_agentes_oeste START 1;

-- SECUENCIAS REGIONALES SINIESTROS
CREATE SEQUENCE seq_siniestros_norte START 1;
CREATE SEQUENCE seq_siniestros_sur START 1;
CREATE SEQUENCE seq_siniestros_este START 1;
CREATE SEQUENCE seq_siniestros_oeste START 1;

-- SECUENCIAS REGIONALES PAGOS
CREATE SEQUENCE seq_pagos_norte START 1;
CREATE SEQUENCE seq_pagos_sur START 1;
CREATE SEQUENCE seq_pagos_este START 1;
CREATE SEQUENCE seq_pagos_oeste START 1;

-- SECUENCIAS REGIONALES AUDITORIAS
CREATE SEQUENCE seq_auditorias_norte START 1;
CREATE SEQUENCE seq_auditorias_sur START 1;
CREATE SEQUENCE seq_auditorias_este START 1;
CREATE SEQUENCE seq_auditorias_oeste START 1;

-- Índice único para evitar que 2 clientes usen el mismo correo
CREATE UNIQUE INDEX idx_clientes_email
ON clientes(email);

-- Índice único para evitar duplicidad de documentos
CREATE UNIQUE INDEX idx_clientes_numero_documento
ON clientes(numero_documento);

-- Índice compuesto para búsquedas combinando tipo y región
CREATE INDEX idx_polizas_tipo_region
ON polizas(tipo_poliza, region);

-- Acelera consultas por estado (Ej.: "En revisión", "Cerrado")
CREATE INDEX idx_siniestros_estado
ON siniestros(estado_siniestro);

-- Acelera consultas por fecha (muy usado en reportes)
CREATE INDEX idx_siniestros_fecha
ON siniestros(fecha_reporte);

--Creación de tablas Clientes. Principal para identificar clientes.
 CREATE TABLE clientes (
    cliente_id SERIAL PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    telefono VARCHAR(20),
    direccion VARCHAR(150),
    pais VARCHAR(50),
    tipo_documento VARCHAR(20),
    numero_documento VARCHAR(30) UNIQUE,
    fecha_registro DATE DEFAULT CURRENT_DATE
);

SELECT * FROM clientes

--Creación de tablas Polizas. Registra cada póliza de seguro que se emite a los clientes.
CREATE TABLE polizas(
    poliza_id SERIAL PRIMARY KEY,
    cliente_id INT NOT NULL,
    tipo_poliza VARCHAR(50) NOT NULL,
    cobertura VARCHAR(100),
    prima_anual DECIMAL(10,2),
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE NOT NULL,
    estado_poliza VARCHAR(20),
    region VARCHAR(50),
    CONSTRAINT fk_poliza_cliente 
        FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id) ON DELETE CASCADE
);

--Creación de tablas Agentes. Guarda los datos de los agentes de seguros que gestionan clientes y pólizas.
CREATE TABLE agentes (
    agente_id SERIAL PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    region VARCHAR(50),
    puesto VARCHAR(50),
    usuario VARCHAR(50) UNIQUE NOT NULL,
    contrasena VARCHAR(100) NOT NULL,
    fecha_ingreso DATE DEFAULT CURRENT_DATE,
    estado VARCHAR(20)
);

--Creación de tablas Siniestros. Registra los accidentes o reclamaciones hechas por los clientes sobre sus pólizas.
CREATE TABLE siniestros (
    siniestro_id SERIAL PRIMARY KEY,
    poliza_id INT NOT NULL,
    fecha_reporte DATE DEFAULT CURRENT_DATE,
    tipo_siniestro VARCHAR(50),
    monto_estimado DECIMAL(10,2),
    estado_siniestro VARCHAR(20),
    descripcion TEXT,
    agente_id INT,
    CONSTRAINT fk_siniestro_poliza 
        FOREIGN KEY (poliza_id) REFERENCES polizas(poliza_id) ON DELETE CASCADE,
    CONSTRAINT fk_siniestro_agente
        FOREIGN KEY (agente_id) REFERENCES agentes(agente_id)
);

--Creación de tablas Pagos. Almacena todos los pagos hechos por los clientes para sus pólizas.
CREATE TABLE pagos (
    pago_id SERIAL PRIMARY KEY,
    poliza_id INT NOT NULL,
    fecha_pago DATE DEFAULT CURRENT_DATE,
    monto DECIMAL(10,2) NOT NULL,
    metodo_pago VARCHAR(30),
    estado_pago VARCHAR(20),
    CONSTRAINT fk_pago_poliza
        FOREIGN KEY (poliza_id) REFERENCES polizas(poliza_id) ON DELETE CASCADE
);

--Creación de tablas Auditorías. Lleva un registro de cambios en las tablas importantes para auditoría interna y control de seguridad.
CREATE TABLE auditorias (
    auditoria_id SERIAL PRIMARY KEY,
    tabla_afectada VARCHAR(50),
    accion VARCHAR(20),
    usuario VARCHAR(50),
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    detalles_cambio TEXT
);

-- Auditoría para pólizas
CREATE TABLE auditoria_polizas (
    id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    poliza_id INT,
    accion VARCHAR(30),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    detalles TEXT
);

-- Auditoría para siniestros
CREATE TABLE auditoria_siniestros (
    id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    siniestro_id INT,
    accion VARCHAR(30),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    detalles TEXT
);

-- Auditoría para pagos
CREATE TABLE auditoria_pagos (
    id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    pago_id INT,
    accion VARCHAR(30),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    detalles TEXT
);

-- Auditoría para errores (registro de excepciones desde SPs / funciones)
CREATE TABLE auditoria_errores (
    id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    procedimiento VARCHAR(200),
    mensaje TEXT,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    datos_adicionales TEXT
);

-- Tabla de alertas (para notificaciones automáticas o cron jobs)
CREATE TABLE alertas (
    alerta_id INT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    poliza_id INT,
    tipo_alerta VARCHAR(50),
    mensaje TEXT,
    creada_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    leida BOOLEAN DEFAULT FALSE
);

--Audito

-- Auditoría para pagos
CREATE TABLE auditoria_pagos (
    id INT IDENTITY PRIMARY KEY,
    pago_id INT,
    accion VARCHAR(30),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    detalles TEXT
);

-- Auditoría para errores
CREATE TABLE auditoria_errores (
    id INT IDENTITY PRIMARY KEY,
    procedimiento VARCHAR(200),
    mensaje TEXT,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    datos_adicionales TEXT
);

-- FUNCIÓN: Validar si la póliza está vigente
-- Se usa antes de INSERT en siniestros
CREATE OR REPLACE FUNCTION validar_vigencia_siniestro()
RETURNS TRIGGER AS $$
DECLARE
    estado_pol VARCHAR(20);
    fin DATE;
BEGIN
    -- Obtenemos estado y fecha final de la póliza
    SELECT estado_poliza, fecha_fin
    INTO estado_pol, fin
    FROM polizas
    WHERE poliza_id = NEW.poliza_id;

    -- Validación 1: póliza activa
    IF estado_pol <> 'Activa' THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('validar_vigencia_siniestro',
                'Error: la póliza no está activa.',
                'poliza_id=' || NEW.poliza_id);
        RAISE EXCEPTION 'No se puede registrar siniestro: póliza no activa.';
    END IF;

    -- Validación 2: póliza vigente en fecha
    IF CURRENT_DATE > fin THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('validar_vigencia_siniestro',
                'Error: la póliza está vencida.',
                'poliza_id=' || NEW.poliza_id);
        RAISE EXCEPTION 'No se puede registrar siniestro: póliza vencida.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


-- Trigger asociado
CREATE TRIGGER trg_validar_siniestro
BEFORE INSERT ON siniestros
FOR EACH ROW
EXECUTE FUNCTION validar_vigencia_siniestro();


-- FUNCIÓN: Validar vigencia antes de registrar un pago
CREATE OR REPLACE FUNCTION validar_vigencia_pago()
RETURNS TRIGGER AS $$
DECLARE
    estado_pol VARCHAR(20);
    fin DATE;
BEGIN
    SELECT estado_poliza, fecha_fin
    INTO estado_pol, fin
    FROM polizas
    WHERE poliza_id = NEW.poliza_id;

    -- Cancelada
    IF estado_pol = 'Cancelada' THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('validar_vigencia_pago',
                'Error: la póliza está cancelada.',
                'poliza_id=' || NEW.poliza_id);
        RAISE EXCEPTION 'Error: la póliza está cancelada.';
    END IF;

    -- Vencida
    IF CURRENT_DATE > fin THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('validar_vigencia_pago',
                'Error: la póliza está vencida.',
                'poliza_id=' || NEW.poliza_id);
        RAISE EXCEPTION 'Error: la póliza está vencida.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_validar_pago
BEFORE INSERT ON pagos
FOR EACH ROW
EXECUTE FUNCTION validar_vigencia_pago();

-- FUNCIÓN: Auditoría para cambios en SINIESTROS
CREATE OR REPLACE FUNCTION auditar_siniestros()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO auditoria_siniestros(siniestro_id, accion, usuario, detalles)
    VALUES(
        COALESCE(NEW.siniestro_id, OLD.siniestro_id),
        TG_OP,                -- INSERT, UPDATE, DELETE
        current_user,
        row_to_json(COALESCE(NEW, OLD))::text
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_auditar_siniestros
AFTER INSERT OR UPDATE OR DELETE ON siniestros
FOR EACH ROW
EXECUTE FUNCTION auditar_siniestros();

-- FUNCIÓN: Auditoría para cambios en PAGOS
CREATE OR REPLACE FUNCTION auditar_pagos()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO auditoria_pagos(pago_id, accion, usuario, detalles)
    VALUES(
        COALESCE(NEW.pago_id, OLD.pago_id),
        TG_OP,
        current_user,
        row_to_json(COALESCE(NEW, OLD))::text
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_auditar_pagos
AFTER INSERT OR UPDATE OR DELETE ON pagos
FOR EACH ROW
EXECUTE FUNCTION auditar_pagos();

-- FUNCIÓN: Actualizar estado tras registrar un pago
CREATE OR REPLACE FUNCTION actualizar_estado_poliza_por_pago()
RETURNS TRIGGER AS $$
BEGIN
    -- Si el pago fue confirmado, la póliza se activa
    IF NEW.estado_pago = 'Confirmado' THEN
        UPDATE polizas
        SET estado_poliza = 'Activa'
        WHERE poliza_id = NEW.poliza_id;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_estado_poliza_pago
AFTER INSERT ON pagos
FOR EACH ROW
EXECUTE FUNCTION actualizar_estado_poliza_por_pago();

-- FUNCIÓN: Suspender póliza cuando se registra un siniestro
CREATE OR REPLACE FUNCTION actualizar_estado_por_siniestro()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE polizas
    SET estado_poliza = 'Suspendida'
    WHERE poliza_id = NEW.poliza_id;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_estado_poliza_siniestro
AFTER INSERT ON siniestros
FOR EACH ROW
EXECUTE FUNCTION actualizar_estado_por_siniestro();

-- FUNCIÓN: Alerta de pólizas próximas a vencer
CREATE OR REPLACE FUNCTION alertar_vencimiento_poliza()
RETURNS TRIGGER AS $$
DECLARE
    dias_rest INT;
BEGIN
    SELECT (fecha_fin - CURRENT_DATE)
    INTO dias_rest
    FROM polizas
    WHERE poliza_id = NEW.poliza_id;

    IF dias_rest <= 30 THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('alertar_vencimiento_poliza',
                'ALERTA: póliza próxima a vencer',
                'poliza_id=' || NEW.poliza_id || ', dias_restantes=' || dias_rest);
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_alerta_vencimiento
AFTER UPDATE OF fecha_fin ON polizas
FOR EACH ROW
EXECUTE FUNCTION alertar_vencimiento_poliza();

-- FUNCIÓN: Alerta por pagos retrasados
CREATE OR REPLACE FUNCTION alertar_pago_retrasado()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.estado_pago = 'Retrasado' THEN
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('alertar_pago_retrasado',
                'ALERTA: pago retrasado',
                'pago_id=' || NEW.pago_id || ', poliza_id=' || NEW.poliza_id);
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_alerta_pago_retrasado
AFTER INSERT OR UPDATE ON pagos
FOR EACH ROW
EXECUTE FUNCTION alertar_pago_retrasado();

-- TRANSACCIÓN: Registrar siniestro + suspender póliza
DO $$
DECLARE
    v_poliza_id INT := 1;          -- Id de la póliza (ejemplo, puedes cambiarlo)
    v_agente_id INT := 1;          -- Agente que reporta el siniestro
    v_siniestro_id INT;            -- Para guardar el id generado
BEGIN
    -- INICIAR TRANSACCIÓN
    BEGIN;

    -- 1. Insertar siniestro
    INSERT INTO siniestros(poliza_id, fecha_reporte, tipo_siniestro, monto_estimado,
                           estado_siniestro, descripcion, agente_id)
    VALUES(v_poliza_id, CURRENT_DATE, 'Accidente', 15000,
           'En revisión', 'Siniestro generado desde transacción', v_agente_id)
    RETURNING siniestro_id INTO v_siniestro_id;

    -- 2. Actualizar estado de póliza (suspender)
    UPDATE polizas
    SET estado_poliza = 'Suspendida'
    WHERE poliza_id = v_poliza_id;

    -- SI TODO SALE BIEN
    COMMIT;

    RAISE NOTICE 'Siniestro registrado y póliza suspendida. ID=%', v_siniestro_id;

EXCEPTION
    WHEN OTHERS THEN
        -- DESHACER TODO SI FALLA
        ROLLBACK;

        -- Registrar error
        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('transaccion_registro_siniestro',
                SQLERRM,
                'poliza_id=' || v_poliza_id);

        RAISE NOTICE 'ERROR: Se revertió la transacción.';
END;
$$;

-- TRANSACCIÓN: Cancelar póliza + revertir sus pagos pendientes
DO $$
DECLARE
    v_poliza_id INT := 3;   -- Ejemplo
    v_pag_id INT;
BEGIN
    BEGIN;

    -- 1. Cambiar estado de póliza
    UPDATE polizas
    SET estado_poliza = 'Cancelada'
    WHERE poliza_id = v_poliza_id;

    -- 2. Buscar pagos pendientes y revertirlos
    UPDATE pagos
    SET estado_pago = 'Revertido'
    WHERE poliza_id = v_poliza_id
      AND estado_pago = 'Pendiente';

    -- Si nada falla
    COMMIT;

    RAISE NOTICE 'Póliza cancelada y pagos pendientes revertidos.';

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;

        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('transaccion_cancelar_poliza',
                SQLERRM,
                'poliza_id=' || v_poliza_id);

        RAISE NOTICE 'ERROR en la cancelación, transacción revertida.';
END;
$$;

-- TRANSACCIÓN: Garantizar integridad entre regiones
DO $$
DECLARE
    v_poliza INT := 5;
BEGIN
    BEGIN;

    -- Región 1 actualiza estado
    UPDATE polizas
    SET region = 'Norte'
    WHERE poliza_id = v_poliza;

    -- Región 2 verifica que no esté cancelada
    IF (SELECT estado_poliza FROM polizas WHERE poliza_id = v_poliza) = 'Cancelada' THEN
        RAISE EXCEPTION 'No se puede sincronizar en región 2 porque la póliza está cancelada';
    END IF;

    COMMIT;

    RAISE NOTICE 'Actualización regional completada.';

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;

        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('transaccion_regiones',
                SQLERRM,
                'poliza=' || v_poliza);

        RAISE NOTICE 'ERROR durante sincronización de regiones.';
END;
$$;

-- Plantilla de manejo de errores (TRY/CATCH)
DO $$
BEGIN
    BEGIN;

    -- Aquí pones cualquier operación crítica
    UPDATE polizas SET estado_poliza = 'Activa'
    WHERE poliza_id = 10;

    -- Confirmar si todo está bien
    COMMIT;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;

        INSERT INTO auditoria_errores(procedimiento, mensaje, datos_adicionales)
        VALUES ('ejemplo_try_catch',
                SQLERRM,
                'poliza_id=10');

        RAISE NOTICE 'Error detectado y registrado correctamente.';
END;
$$;

-- PROCEDIMIENTO: Registrar nueva póliza
CREATE OR REPLACE FUNCTION registrar_poliza(
    p_cliente_id INT,
    p_tipo_poliza VARCHAR,
    p_monto DECIMAL,
    p_region VARCHAR
)
RETURNS VOID AS $$
DECLARE
    nueva_poliza_id INT;
BEGIN
    -- Validar que el cliente exista
    IF NOT EXISTS (SELECT 1 FROM clientes WHERE id = p_cliente_id) THEN
        RAISE EXCEPTION 'El cliente % no existe', p_cliente_id;
    END IF;

    -- Insertar la póliza
    INSERT INTO polizas (cliente_id, tipo, monto, region, estado)
    VALUES (p_cliente_id, p_tipo_poliza, p_monto, p_region, 'ACTIVA')
    RETURNING id INTO nueva_poliza_id;

    -- Registrar auditoría
    INSERT INTO auditoria_pagos (pago_id, accion, usuario, detalles)
    VALUES (NULL, 'REGISTRO_POLIZA', current_user, 
            CONCAT('Se registró la póliza ', nueva_poliza_id));

EXCEPTION
    WHEN OTHERS THEN
        -- Auditoría de errores
        INSERT INTO auditoria_errores (procedimiento, mensaje, datos_adicionales)
        VALUES ('registrar_poliza', SQLERRM, CONCAT('Cliente: ', p_cliente_id));

        RAISE;
END;
$$ LANGUAGE plpgsql;

-- PROCEDIMIENTO: Consultar historial de siniestros
CREATE OR REPLACE FUNCTION historial_siniestros(
    p_cliente_id INT DEFAULT NULL,
    p_pais VARCHAR DEFAULT NULL,
    p_tipo_poliza VARCHAR DEFAULT NULL
)
RETURNS TABLE(
    siniestro_id INT,
    cliente VARCHAR,
    tipo_poliza VARCHAR,
    pais VARCHAR,
    fecha DATE,
    descripcion TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        s.id,
        c.nombre,
        p.tipo,
        p.region,
        s.fecha,
        s.descripcion
    FROM siniestros s
    INNER JOIN polizas p ON p.id = s.poliza_id
    INNER JOIN clientes c ON c.id = p.cliente_id
    WHERE (p_cliente_id IS NULL OR c.id = p_cliente_id)
      AND (p_pais IS NULL OR p.region = p_pais)
      AND (p_tipo_poliza IS NULL OR p.tipo = p_tipo_poliza);
END;
$$ LANGUAGE plpgsql;

-- PROCEDIMIENTO: Reporte de desempeño regional
CREATE OR REPLACE FUNCTION reporte_regional(p_region VARCHAR)
RETURNS TABLE(
    region VARCHAR,
    total_polizas INT,
    total_siniestros INT,
    monto_total DECIMAL
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        p_region,
        (SELECT COUNT(*) FROM polizas WHERE region = p_region),
        (SELECT COUNT(*) FROM siniestros s 
            INNER JOIN polizas pp ON pp.id = s.poliza_id 
            WHERE pp.region = p_region),
        (SELECT SUM(monto) FROM polizas WHERE region = p_region);
END;
$$ LANGUAGE plpgsql;

-- PROCEDIMIENTO: Monto total asegurado
CREATE OR REPLACE FUNCTION monto_asegurado(
    p_region VARCHAR DEFAULT NULL,
    p_tipo VARCHAR DEFAULT NULL
)
RETURNS DECIMAL AS $$
DECLARE
    total DECIMAL;
BEGIN
    SELECT SUM(monto)
    INTO total
    FROM polizas
    WHERE (p_region IS NULL OR region = p_region)
      AND (p_tipo IS NULL OR tipo = p_tipo);

    RETURN COALESCE(total, 0);
END;
$$ LANGUAGE plpgsql;

-- PROCEDIMIENTO: Aplicar descuentos
CREATE OR REPLACE FUNCTION aplicar_descuentos(
    p_region VARCHAR,
    p_descuento DECIMAL
)
RETURNS VOID AS $$
BEGIN
    -- Aplicar descuento solo a pólizas activas de esa región
    UPDATE polizas
    SET monto = monto - (monto * (p_descuento / 100))
    WHERE region = p_region
      AND estado = 'ACTIVA';

    -- Auditoría
    INSERT INTO auditoria_pagos (pago_id, accion, usuario, detalles)
    VALUES (NULL, 'DESCUENTO_REGION', current_user,
            CONCAT('Se aplicó ', p_descuento, '% a región ', p_region));
END;
$$ LANGUAGE plpgsql;
