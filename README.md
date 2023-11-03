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

