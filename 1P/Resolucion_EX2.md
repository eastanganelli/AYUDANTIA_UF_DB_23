# Ejercicio 2
## MR
- ELABORA (laboratorio, droga)
- PRODUCE (laboratorio, remedio)
- VENDE (farmacia, remedio, precio)
- COMPUESTO_POR (remedio, droga)
- RECETA (CodBarras, fecha, medico.matricula, paciente)
- RECETADOS (CodBarras, remedio)
- MEDICO (matricula, nombre, apellido, dirección, localidad, teléfono)
- REMEDIO (nombre, laboratorio, precio)
---
### <i>a.</i> Listar las drogas que componen los remedios recetados durante todo el mes de enero de 2023 por el doctor Julián Álvarez, matricula 123456.
<b>SQL</b>
```SQL
select
	COMPUESTO_POR.drogaNom
from RECETA
join RECETADOS on RECETADOS.codBarras=RECETA.codBarras
join COMPUESTO_POR on (COMPUESTO_POR.remedioNom=RECETADOS.remedioNom and COMPUESTO_POR.remedioLab=RECETADOS.remedioLab)
where RECETA.medMat like '123456' and (RECETA.fecha>='01-01-2023' and RECETA.fecha<='31-01-2023');
```
<b>Algebra Relacional</b>
```txt
π COMPUESTO_POR.drogaNom σ RECETA.medMat like '123456' and (RECETA.fecha ≥ '01-01-2023' and RECETA.fecha ≤ '31-01-2023') ( ( RECETA ⨝ RECETADOS.codBarras = RECETA.codBarras RECETADOS ) ⨝ (COMPUESTO_POR.remedioNom = RECETADOS.remedioNom and COMPUESTO_POR.remedioLab = RECETADOS.remedioLab) COMPUESTO_POR )
```
<br>

### <i>b.</i> Mostrar la farmacia que vendió más unidades de GASTEC que las demás.
<b>SQL</b>
```SQL
select
	VENDE.farmacia,
	count(*) as Cantidad
from VENDE
where VENDE.remedioNom like 'Gastec'
group by VENDE.farmacia,VENDE.remedioNom,VENDE.remedioLab
order by Cantidad DESC
```
<b>Algebra Relacional</b>
```
τ Cantidad desc π VENDE.farmacia, Cantidad γ VENDE.farmacia, VENDE.remedioNom, VENDE.remedioLab; COUNT(*)→Cantidad σ VENDE.remedioNom like 'Gastec' VENDE
```
<br>

### <i>c.</i> Listar los remedios cuyos precios son mayores al remedio más vendido en FARMACITY.
<b>SQL</b>
```sql
select
	VENDE.remedioNom,
	VENDE.remedioLab,
	VENDE.precio
from VENDE, (
			select
				VENDE.remedioNom,
				VENDE.remedioLab,
				count(*) as cantidad,
				avg(VENDE.precio) as precio
			from VENDE
			where VENDE.farmacia like '%Farmacity%'
			group by VENDE.remedioNom, VENDE.remedioLab
			order by cantidad Desc
			limit 1
		) as R
where VENDE.precio > R.precio
```
<b>Algebra Relacional</b>
```
MasVendido = ρ R ( σ ROWNUM() > 0 and ROWNUM() ≤ 1 τ cantidad desc π VENDE.remedioNom, VENDE.remedioLab, cantidad, precio γ VENDE.remedioNom, VENDE.remedioLab; COUNT(*)→cantidad, AVG(VENDE.precio)→precio σ VENDE.farmacia like '%Farmacity%' VENDE )

π VENDE.remedioNom, VENDE.remedioLab, VENDE.precio σ VENDE.precio > R.precio ( VENDE ⨯ MasVendido )
```
<br>

### <i>d.</i> Mostrar el remedio más caro y el más barato elaborados con la droga METFORMINA.
<b>SQL</b>
```sql
(
	select
		RemMax.remedio,
		RemMax.laboratorio,
		RemMax.precio
	from REMEDIO as RemMax
	join COMPUESTO_POR on (COMPUESTO_POR.remedioNom=RemMax.remedio and COMPUESTO_POR.remedioLab=RemMax.laboratorio)
	where COMPUESTO_POR.drogaNom = 'metformina'
	order by RemMax.precio DESC
	limit 0,1
)
UNION
(
	select
		RemMin.remedio,
		RemMin.laboratorio,
		RemMin.precio
	from REMEDIO as RemMin
	join COMPUESTO_POR on (COMPUESTO_POR.remedioNom=RemMin.remedio and COMPUESTO_POR.remedioLab=RemMin.laboratorio)
	where COMPUESTO_POR.drogaNom = 'metformina'
	order by RemMin.precio
	limit 0,1
)
```
<b>Algebra Relacional</b>
```
queryMax= σ ROWNUM() > 0 and ROWNUM() ≤ 1 τ RemMax.precio desc π RemMax.remedio, RemMax.laboratorio, RemMax.precio σ COMPUESTO_POR.drogaNom = 'metformina' ( ρ RemMax REMEDIO ⨝ (COMPUESTO_POR.remedioNom = RemMax.remedio and COMPUESTO_POR.remedioLab = RemMax.laboratorio) COMPUESTO_POR )

queryMin= σ ROWNUM() > 0 and ROWNUM() ≤ 1 τ RemMin.precio asc π RemMin.remedio, RemMin.laboratorio, RemMin.precio σ COMPUESTO_POR.drogaNom = 'metformina' ( ρ RemMin REMEDIO ⨝ (COMPUESTO_POR.remedioNom = RemMin.remedio and COMPUESTO_POR.remedioLab = RemMin.laboratorio) COMPUESTO_POR )

queryMax ∪ queryMin
```
<br>

### <i>e.</i> Listar los laboratorios que no elaboran ninguna de las drogas de los remedios que producen.
<b>SQL</b>
```sql
select distinct
	CP1.remedioLab
FROM COMPUESTO_POR as CP1
LEFT OUTER JOIN COMPUESTO_POR as CP2 on CP2.drogaLab=CP1.remedioLab
where CP2.remedioLab is null
```
<b>Algebra Relacional</b>
```
π CP1.remedioLab σ CP2.remedioLab = null ( ρ CP1 COMPUESTO_POR ⟕ CP2.drogaLab = CP1.remedioLab ρ CP2 COMPUESTO_POR )
```