# F1WorldChampioship
<img src="https://github.com/ManuHernandezz/F1WorldChampioship/assets/130862652/c8e02840-aa3b-431d-8c1f-dc97f4d4b8bf" width="150" height="150">


This SQL project involves various queries analyzing Formula 1 data, including race results, driver standings, and constructor performances.

### 1)¿Cuántos circuitos hay en cada país?
    
    SELECT country, COUNT(*) AS Q_circuits
    FROM circuits
    GROUP BY country
    ORDER BY Q_circuits DESC

### 2) Lista los 5 constructores con más puntos en una temporada específica.

    SELECT SUM(r.points) AS total_points, c.name
    FROM results r
    LEFT JOIN constructors c
      ON r.constructorId = c.constructorId
    LEFT JOIN races ra
      ON ra.raceId = r.raceId
    WHERE ra.year = 1985
    GROUP BY c.name
    ORDER BY total_points DESC

### 3) Encuentra todos los pilotos que han ganado al menos una carrera.

    SELECT DISTINCT(d.surname)
    FROM drivers d
    LEFT JOIN results r
      ON r.driverId = d.driverId
    WHERE r.position = 1
    LIMIT 10 


### 4) ¿Cuál es el constructor con más victorias en una temporada específica?
  
  --ES VICTORIAAS NO PUNTOS OJO, OSEA QUE SALGA PRIMERO
  
    SELECT c.name, COUNT(*) AS total_victories
    FROM constructors c
    JOIN results r ON c.constructorId = r.constructorId
    JOIN races ra ON r.raceId = ra.raceId
    WHERE ra.year = 1984 AND r.position = 1
    GROUP BY c.name
    ORDER BY total_victories DESC;

### 5) Encuentra los pilotos que nunca han sido descalificados.

    SELECT DISTINCT (d.surname || ', ' || d.forename) AS Names
    FROM drivers d
    WHERE d.driverId NOT IN
      (
        SELECT DISTINCT r.driverId
        FROM results r
        WHERE r.statusId = 2
      )
    ORDER BY Names DESC

### 6) ¿Cuántas carreras ha ganado cada piloto en su carrera?
    
  --contar el numero de carreras qeu gano cada piloto, para esto 
  --position -->results (contar por driver id where position = 1)    --forename, surname --> drivers
  --unir con driverID
  
    SELECT d.forename, d.surname, COUNT(r.position)
    FROM drivers d
    INNER JOIN  results r
      ON d.driverID = r.driverID
    WHERE position = 1
    GROUP BY d.driverID

### 7) Encuentra la carrera con la mayor cantidad de abandonos.

  --contar lA cantidad de status <> a 1 , que es finished.  --> results
  --relacionarlo ccon la tabla status para ver cual es el finished
  --relacionarlo con la tabla races para ver el año y el nombre de la carrera
    
    SELECT r.name, r.year, COUNT(re.statusId) AS Q_abandoned
    FROM races r
    INNER JOIN results re
      ON r.raceId = re.raceId
    INNER JOIN status s
      ON re.statusId = s.statusId
    WHERE s.status <>  '%finished%'
    GROUP BY r.name , r.year
    ORDER BY Q_abandoned DESC
    LIMIT 1



### 8) Encuentra todos los pilotos cuyo apellido comience con 'A'.
  
  --uso la tabla drivers, Forename

    SELECT forename
    FROM drivers
    WHERE forename like 'A%'
    ORDER BY forename DESC

### 9) Lista los circuitos cuyos nombres contengan la palabra 'Grand'.
  --muy similar al anterior, pero esta  vez uso la tabla circuits, name LIKE

    SELECT name
    FROM circuits
    WHERE name LIKE '%Grand%'

### 10) Busca todos los resultados de carreras donde el estado del piloto contenga la palabra 'engine'.

  --uso status -->tabla status para buscar la palabra engine
  --tabla resultas -> uso raceid para relacionarlo con races, y statusid para relaacionarlo con status
  --races-> uso la tabla name y year tambien
  --Podria hacer un contador

    SELECT r.name, r.year, COUNT(re.statusId) AS count_finished, re.statusId
    FROM races r
    INNER JOIN results re
      ON r.raceId = re.raceId
    INNER JOIN status s
      ON s.statusId = re.statusId
    WHERE s.status LIKE '%engine%'
    GROUP BY  r.name, r.year
    ORDER BY r.year ASC


### 11) Muestra la evolución de los puntos de un piloto a lo largo de una temporada.

  --results -> points, driverID, raceId
  --races -> raceId, year
  --driver -> driverID, surname

    SELECT d.surname,
         ra.year,
         r.points,
         SUM(r.points) OVER (PARTITION BY r.driverId ORDER BY ra.year, ra.raceId) AS Total_points
    FROM results r
    INNER JOIN races ra 
	    ON r.raceId = ra.raceId
    INNER JOIN drivers d 
	    ON d.driverId = r.driverId
    WHERE ra.year = 2020
      AND r.driverId = 1
    ORDER BY r.driverId, ra.year, ra.raceId

### 12) Lista las 10 vueltas más rápidas en un circuito específico.
  
    SELECT r.fastestLapSpeed, ra.name
    FROM results r
    INNER JOIN races ra
      ON r.raceId = ra.raceId
    WHERE ra.name LIKE '%monaco%'
      AND r.fastestLapSpeed <> '\N'
    ORDER BY r.fastestLapSpeed DESC
    LIMIT 10

### 13) Utiliza un JOIN para mostrar los nombres de los pilotos junto con los nombres de sus constructores.
    
  --drivers -> driverid, surname, forename
  --constructors-> constructorId, name
  --results -> driverId, constructorId

    SELECT DISTINCT (d.forename|| ', ' || d.surname) AS Driver_name, 
                c.name AS constructor_name
    FROM drivers d
    INNER JOIN results r
      ON d.driverId = r.driverId
    INNER JOIN constructors c
      ON r.constructorId = c.constructorId 
	
### 14) Muestra un ranking de pilotos basado en la cantidad de primeros lugares obtenidos.
  
  --hacer un contador ordenado desc de los primeros puestos
  --results -> position, driverdId
  --drivers -> driverId, Surname, forename

    SELECT (d.forename || ', ' || d.surname) AS driver_name,
          COUNT(r.position) AS Q_first_place
    FROM drivers d
    INNER JOIN results r
      ON d.driverId = r.driverId
    WHERE r.position = 1
    GROUP BY driver_name
    ORDER BY Q_first_place DESC

### 15) Utiliza un LEFT JOIN para listar todos los pilotos y las carreras que no han terminado.

  -- status-> statusid , status NOT LIKE '%finish%'
  -- results -> raceId , driver ID, statusId
  -- drivers -> driverdId, forename, surname
  -- races -> raceId, year, name

    SELECT  (d.forename || ', ' || d.surname) AS driver_name, 
          s.status,         
          ra.year,
          ra.name
    FROM status s
    LEFT JOIN results r
      ON s.statusId = r.statusId
    LEFT JOIN drivers d
      ON r.driverId = d.driverId
    LEFT JOIN races ra
      ON r.raceId = ra.raceId
    WHERE s.status NOT LIKE '%finish%'

### 16) ¿Cuántas veces un piloto ha quedado en la misma posición que su número de coche?
  
  --hacer un count position  y luego un where donde sea igual al numero del piloto
  --results -> driverId, position
  -- drivers -> driverdId, forename, Surname, NUMBER

    SELECT  (d.forename || ', ' || d.surname) AS driver_name, 
          d.number,
          COUNT(r.position) AS Q_position_number
    FROM drivers d
    INNER JOIN results r
      ON d.driverId = r.driverId 
    GROUP BY driver_name, d.number
    HAVING Q_position_number = d.number;


### 17) Muestra la clasificación de constructores en una temporada específica, con un desglose de puntos por piloto.
  
  --results -> points, driverId, constructorId, raceId
  --drivers -> driverid, forename, surname
  --constructors -> constructorId, name 
  --races -> raceId, year

    SELECT  c.name AS constructor_name, 
          (d.forename || ', ' || d.surname) AS driver_name, 
          ra.year,
         SUM(r.points) AS total_points  
    FROM constructors c
    INNER JOIN results r
      ON c.constructorId = r.constructorId
    INNER JOIN  drivers d
      ON d.driverId = r.driverId
    INNER JOIN races ra
      ON r.raceId = ra.raceId
    WHERE ra.year = 1987
    GROUP BY constructor_name , driver_name
    ORDER BY total_points DESC

### 18) Encuentra el circuito en el que cada constructor ha logrado su mejor resultado.
  
  --results -> fastestLapTime, constructorId, driverdId, raceId
  --races -> raceId, year, name
  --drivers -> driverId, forename, surname
  --constructors -> constructorId, name
  --MIN fastestelaptime

    SELECT  c.name AS consturctor_name,
          (d.forename || ', ' || d.surname) AS driver_name, 
          ra.name AS circuit_name,
          ra.year,
          MIN(r.fastestLapTime) AS min_lap_time
    FROM results r
    INNER  JOIN drivers d
      ON d.driverId = r.driverId
    INNER JOIN races ra 
      ON ra.raceId = r.raceId
    INNER JOIN constructors c
      ON c.constructorId = r.constructorId
    WHERE (r.fastestLapTime) <> '\N'
    GROUP BY consturctor_name
    ORDER BY min_lap_time ASC

### 19) Muestra la progresión anual de puntos de un constructor específico.
  
  --usar partition by para que se acumule el resultado
  -- constructors -> name, constructorId
  --results -> constructorId, raceId, points
  --races -> raceId, year
  --nomas se agrupa por constructorId

    SELECT c.name,
           ra.year,
         r.points,
         SUM(r.points) OVER (PARTITION BY r.constructorId, c.name ORDER BY ra.year, ra.raceID) AS Cumulative_points
    FROM results r
    INNER JOIN races ra 
      ON r.raceId = ra.raceId
    INNER JOIN constructors c
      ON c.constructorId = r.constructorId
    WHERE c.name LIKE 'ferrari'
    GROUP BY ra.year, c.name
    ORDER BY ra.year ASC;


### 20) ¿Cuál es el promedio de paradas en boxes por carrera para cada piloto? (lo voy hacer de un solo año; o veo de hacerlo por un piloto)

  --avg (miliseconds) /1000 AS segundos 
  --pit_stops -> miliseconds; driverId, raceId
  --drivers -> driverdId, concat, forename, Surname
  --races -> raceId, year

    SELECT (d.forename || ', ' || d.surname) AS driver_name,
          ROUND(AVG(p.milliseconds/10000),2) AS Pit_seconds
    FROM drivers d
    INNER JOIN pit_stops p
      ON d.driverId = p.driverId
    INNER JOIN results r
      ON d.driverId = r.driverId
    INNER JOIN races ra
      ON r.raceId = ra.raceId
    WHERE (d.forename || ', ' || d.surname) LIKE '%alonso%'



### 21) Utiliza CTE para listar los pilotos junto con el total de puntos obtenidos en cada circuito a lo largo de sus carreras.
  
  -- drivers-> driverdId, surname, forename
  -- races -> raceId, year, name. 
  -- results -> raceId, driverdId, points. 

    WITH DriverPoints  AS 
      (   SELECT SUM(points) AS sum__points,
                  driverId,
                  raceId
          FROM results
          GROUP BY raceId, driverId
      )

    SELECT (d.forename || ', ' || d.surname) AS driver_name,
          ra.name,
          ra.year,
          dp.sum__points
    FROM drivers d
    INNER JOIN DriverPoints dp
      ON dp.driverId = d.driverId
    INNER JOIN races ra
      ON ra.raceId = dp.raceId
    WHERE (d.forename || ', ' || d.surname) LIKE '%fangio%'
