# Teste-bd


CREATE TABLE IF NOT EXISTS pessoa (
    codigo INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(45),
    email VARCHAR(45),
    telefone VARCHAR(45)
);


CREATE TABLE IF NOT EXISTS historico (
    codigo INT AUTO_INCREMENT PRIMARY KEY,
    codigo_usuario_sistema INT,
    evento VARCHAR(45),
    tabela VARCHAR(45),
    chave INT,
    comando_sql TEXT,
    data_hora DATETIME
);


DELIMITER $$

DROP PROCEDURE IF EXISTS SQLEXEC $$
CREATE PROCEDURE SQLEXEC(IN comando_sql TEXT, IN cod_usuario_sistema INT, IN tabela VARCHAR(45), IN chave INT, OUT cod_retorno TINYINT)
BEGIN
    DECLARE aux_str TEXT;
    DECLARE aux_chave INT;
	DECLARE evento TEXT;
	DECLARE EXIT HANDLER FOR SQLEXCEPTION
	  BEGIN
   	     SET cod_retorno=1;
		 ROLLBACK;
      END;
	DECLARE EXIT HANDLER FOR SQLWARNING
	  BEGIN
		SET cod_retorno=2;
		ROLLBACK;
	  END;
	START TRANSACTION;
		SET @S=COMANDO_SQL;
		PREPARE STMT FROM @S;
		EXECUTE STMT;
		DEALLOCATE PREPARE STMT;
		SET aux_str = UPPER(comando_sql);
		SELECT SUBSTRING_INDEX(aux_str, ' ', 1) INTO evento;
        	IF (chave=-1) then
                SELECT AUTO_INCREMENT into aux_chave FROM information_schema.tables
 WHERE table_name = 'pessoa' AND table_schema = 'tads22_geremias';

                SET chave=aux_chave-1; 
		END IF;
  		INSERT INTO historico(codigo_usuario_sistema,evento,tabela,chave,data_hora,comando_sql) VALUES (cod_usuario_sistema, evento, lower(tabela), chave, now(), comando_sql);
		SET cod_retorno=0;
	COMMIT;
END $$
DELIMITER ;


call sqlexec('insert into pessoa(nome,email,telefone) values (\'Joao da Silva\',\'joao@gmail.com\',\'123123\')', 1, 'pessoa', -1, @retorno);
select @retorno;

DROP PROCEDURE IF EXISTS SQLEXEC;
drop table pessoa;
drop table historico;
