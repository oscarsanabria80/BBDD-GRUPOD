# BBDD-Hotel Rural
1.Realiza una funcion ComprobarPago que reciba como parametros un codigo de cliente y un codigo de actividad y devuelva un TRUE si el cliente ha pagado la ultima actividad con ese 
codigo que ha realizado y un FALSE en caso contrario. Debes controlar las siguientes excepciones:Cliente inexistente, Actividad Inexistente, Actividad realizada en regimen de Todo Incluido
y El cliente nunca ha realizado esa actividad.

- Cliente inexistente.

<pre>
create or replace procedure Clienteinexistente (p_clientenif personas.NIF%type)
IS 

    v_codcliente number;

begin

    select COUNT(*) INTO v_codcliente
    FROM personas
    where NIF = p_clientenif;
    if v_codcliente=0 then 
        RAISE_APPLICATION_ERROR(-20001, "ESTE CLIENTE NO EXISTE")
    end if;
end;
/
</pre>

- Actividad Inexistente

<pre>
create or replace procedure actividadinexistente (p_actividad actividades.codigoQ%type)
IS 

    v_codactividad number;

begin

    select COUNT(*) INTO v_codactividad
    FROM actividades
    where codigo = p_actividad;
    if v_codactividad=0 then 
        RAISE_APPLICATION_ERROR(-20002, "ESTA ACTIVIDAD NO EXISTE")
    end if;
end;
/
</pre>

- Todo incluido

<pre>
CREATE OR REPLACE PROCEDURE todoincluido (v_codActividad actividades.codigo%type)
IS
    v_todoincluido NUMBER;
BEGIN 
    SELECT COUNT(*)
    INTO v_todoincluido
    FROM actividadesrealizadas
    WHERE codigoestancia IN (SELECT MAX(codigo) FROM estancias WHERE codigoregimen = 'TI')
    AND codigoactividad = v_codActividad;

    IF v_todoincluido > 0 THEN 
        dbms_output.put_line('True');
    ELSE
        dbms_output.put_line('False');
    END IF;
END;
/
</pre>



- El cliente nunca ha realizado esa actividad.
<pre>
create or replace procedure noactividad (p_nif personas.NIF%type, p_codigo actividadesrealizadas.codigoactividad%type)
IS
    v_cliente number;
begin
    select COUNT(*) into v_cliente
    FROM estancias where NIFCliente = p_nif and 
    codigo in (select codigoestancia from actividadesrealizadas where codigoactividad=p_codigo);
    if v_cliente > 0 then
        dbms_output.put_line('Si ha realizado esta actividad');
    ELSE
        dbms_output.put_line('El cliente no ha realizado nunca esta actividad');
    END IF;
end; 
/
</pre>


- Agrupal excepciones
<pre>
CREATE OR REPLACE PROCEDURE Excepciones (p_clin personas.NIF%type,p_codac actividades.codigo%type)
IS
BEGIN
    Clienteinexistente(p_clin);
    actividadinexistente(p_codac);
    todoincluido(p_codac);
    noactividad(p_clin, p_codac);
END;
/
</pre>


- Actividad abonada 

<pre>
CREATE OR REPLACE PROCEDURE A_Abonada (p_codcl personas.nif%type,
p_codacti actividades.codigo%type)
IS
    CURSOR c_abonada IS
    SELECT *
    FROM actividadesrealizadas
    WHERE codigoestancia = (SELECT codigo FROM estancias WHERE
    nifcliente=p_codcl) AND codigoactividad=p_codacti
    ORDER BY fecha DESC
    FETCH FIRST 1 ROWS ONLY;
    v_actabo actividadesrealizadas%ROWTYPE;
BEGIN
    OPEN c_abonada;
    FETCH c_abonada INTO v_actabo;
    IF v_actabo.abonado = 'N' THEN

    DBMS_OUTPUT.PUT_LINE('FALSE');
    ELSE
    DBMS_OUTPUT.PUT_LINE('TRUE');
    END IF;

    CLOSE c_abonada;

END;
/
</pre>

- Función.

<pre>
CREATE OR REPLACE FUNCTION VerificarPago (p_codcliente personas.NIF%type, p_codactividad
actividades.codigo%type)
RETURN BOOLEAN
IS
    v_cliente BOOLEAN;
BEGIN
    Excepciones(v_codcliente, v_codactividad);
    A_Abonada(v_codcliente, v_codactividad);
    RETURN v_cliente;
END;
/
</pre>
