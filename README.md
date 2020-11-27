# XMLIntoMySQLStoredProcedure
Importing XML Data into MySQL Tables Using a Stored Procedure


CREATE DATABASE xml_read;

USE xml_read;

CREATE  TABLE `applicants` (   
    `id`         INT          NOT NULL ,
    `last_name`  VARCHAR(100) NOT NULL ,
    `first_name` VARCHAR(100) NULL ,   
 PRIMARY KEY (`id`) );
 
DELIMITER $$
CREATE PROCEDURE `import_applicant_xml`(xml TEXT, node VARCHAR(255))
BEGIN
    DECLARE xml_content TEXT;
    DECLARE v_row_index INT UNSIGNED DEFAULT 0;   
    DECLARE v_row_count INT UNSIGNED;  
    DECLARE v_xpath_row VARCHAR(255); 
 
    SET xml_content = xml;
 
    -- calculate the number of row elements.   
    SET v_row_count  = EXTRACTVALUE(xml_content, CONCAT('count(', node, ')')); 
    
    -- loop through all the row elements    
    WHILE v_row_index < v_row_count DO                
        SET v_row_index = v_row_index + 1;        
        SET v_xpath_row = CONCAT(node, '[', v_row_index, ']/@*');
        INSERT INTO applicants VALUES (
            EXTRACTVALUE(xml_content, CONCAT(v_xpath_row, '[1]')),
            EXTRACTVALUE(xml_content, CONCAT(v_xpath_row, '[2]')),
            EXTRACTVALUE(xml_content, CONCAT(v_xpath_row, '[3]'))
        );
    END WHILE;
END

SET @data:='<?xml version="1.0"?>
<applicant_list>
  <applicant id="1" fname="Rob" lname="Gravelle"/>
  <applicant id="2" fname="Al" lname="Bundy"/>
  <applicant id="3" fname="Little" lname="Richard"/>
</applicant_list>';

SELECT @data;

CALL import_applicant_xml(@data,'/applicant_list/applicant');
