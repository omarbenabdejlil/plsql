# All plsql exercices are here ! 

<center>
  <div style="align : center;"> <img src="https://img.icons8.com/plasticine/452/oracle-pl-sql--v3.png" class="center"></div>
</center>

## Télecharger les instructions de création des tables ici : ![Lien!](https://iset.uvt.tn/mod/resource/view.php?id=426236) 


## Résumé PLSQL :  ![Lien!](https://iset.uvt.tn/mod/resource/view.php?id=454040) 

<hr>

### Correction de l'exercice numero 1  : ![Lien!](https://ibb.co/r69ky1z)

1)
```sql
select count(*)
  2  from client
  3  ;
select ville, count(*)
  2  from client
  3  group by ville;
```

2)
```sql

select SUM(QteV*prixV)
  2  from vente, Produit
  3  where Vente.NP=Produit.NP;
```
3)

```sql
select SUM(QteV*prixV)
  2  from Vente natural join Produit
  3  GROUP BY NM;

2éme methode
select SUM(QteV*prixV) CA , NM
  2  from Vente join Produit on Vente.NP=Produit.NP
  3  GROUP BY NM;
```
4) 
```sql
select  NM , Nclt , SUM(QteV*prixV) CA
  2    from Vente, Produit
  3  where Vente.NP=Produit.NP
  4  and dateV between '01/01/2020' and '31/12/2020'
  5  group by  NM,Nclt;

2éme méthode :
select  NM , Nclt , SUM(QteV*prixV) CA
  2    from Vente, Produit
  3  where Vente.NP=Produit.NP
  4  and dateV > '01/01/2020' and  dateV <'31/12/2020'
  5  group by  NM,Nclt;
  
3éme methode :
select  NM , Nclt , SUM(QteV*prixV) CA
  2    from Vente, Produit
  3  where Vente.NP=Produit.NP
  4  and extract (year from dateV) = '2020'
  5  group by  NM,Nclt;
 
#4éme methode xD : 
select  NM , Nclt , SUM(QteV*prixV) CA
  2    from Vente, Produit
  3  where Vente.NP=Produit.NP
  4  and TO_CHAR(dateV,'yyyy')='2020'
  5  group by  NM,Nclt;
```

5)
```sql
select  extract(day from dateV) jour , SUM(QteV*prixV) CA from Vente, Produit where (Vente.NP=Produit.NP and dateV > '01/01/2020' and  dateV <'31/12/2020')  group by extract(day from dateV);
```

6)
```sql 
#auto jointure
select M1.NM, M2.NM, M1.ville from magasin M1, magasin M2 where (M1.ville = M2.ville) and (M1.NM < M2.NM);
```
7)
```sql
#-----------creation vu 1 :
 create view V1 as (select NP, SUM(QteV*prixV) caj2019 from Vente natural join Produit where DateV between '01/01/2019' and '31/12/2019' group by NP);

#-------- creation vu 2 : 
create view V2 as (select NP, SUM(QteV*prixV) caj2019 from Vente natural join Produit where DateV between '01/01/2020' and '31/12/2020' group by NP);
//-------- select rows : 
select V1.NP, V1.caj2019, V2.caj2019 from V1 full join V2 ON V1.NP=V2.NP;
```
8)
```sql

 select NP from Vente group by NP having count(distinct Nclt )=(select count(*) from client);

exemple de "Produits achetees par touts les clients" )
select *  from Produit where not exists(select * from client where not exists ( select * from Vente where Vente.NP=Produit.NP and Vente.Nclt=client.Nclt));
```

9)
```sql
# les client qui ont visitées tt les magazins : 
 select Nclt from Vente group by Nclt having count(distinct NM)=(select count(*) from magasin);

//---------autre requetet "kif kif " 
 select * from client where not exists(select * from magasin where not exists(select * from vente where vente.Nclt=client.Nclt and Vente.NM=magasin.NM));
```

10)
```sql
# les clients qui achètent des magasins de Tunis : 
select distinct Nclt from Vente where NM in (select NM from magasin where ville ='Tunis');
```
11)
```sql
les clients qui n’achètent que de magasins de Tunis :  
select distinct Nclt from Vente minus ( select distinct Nclt from Vente where NM in (select NM from magasin where ville <>'Tunis' ));
```

16)
```sql
le chiffre d’affaire par client par magasin ainsi que le chiffre d’affaire par client: 
select NM, Nclt, SUM(Qtev*prixV) CA from vente natural join produit group by Rollup(NM,Nclt);

# ou bien : 
 select NM, NP, Nclt, SUM(Qtev*prixV) CA from vente natural join produit group by Rollup(NM,NP,Nclt);  
 ```
 <hr>
 
 ### Correction du Tp PLSQL Fonction & Procédures : 
 
- Exercice 1 :
> Écrire une fonction qui effectue la conversion des dinars en euros. Utilisez cette fonction pour afficher les prix de vente des prduits en dinars et en euros.

```sql
create or replace function conversion(Montant NUMBER) return number is
Taux number:=3.2;
begin 
return Montant/Taux
end;
/
select NP, Designation, PRIXV ,conversion(PRIXV) prixeuro2  from Produit;
```
- Exercice 2 :
> Ecrire une fonction PL/SQL Calc_App (NumP  In number), qui calcule la quantité d’approvisionnement (Qte_app) d’un produit donné. Cette quantité est calculée suivant la formule suivante :
SI (Qtot/QteStock) > 10 alors Qte_App = (Qtot - QteStk) * 3
Sinon Qte_App = Qtot * 2 avec : 
- Qtot = somme QteV : La somme des quantités vendu de l’article donné.
- QteStock est la quantité en stock de l’article donné (depuis la table Produit) 
- `La fonction développée sera appelée par un programme PL/SQL qui lui fournit tous les articles dont le stock est faible (QteStock <= 5). Afficher les tuples (NP, Designation, Qte_App, Prix_App). Prix_App = PrixA * Qte_App`

```sql
create or replace function Calc_App(NP VARCHAR) return NUMBER is
    Qtot number;
   QteApp number:=0;
    QtStk number;

    begin

   select sum(QteV) into Qtot from Vente where Vente.NP=NP;

    select QteStock into QtStk from Produit where Produit.NP=NP;

   if (Qtot/Qtstk) > 10 then
   QteApp:=(Qtot-Qtstk)*3;
   else
   QteApp:=Qtot*2;
   end if;
   return QteApp;

  end;
  /
declare cursor  c is
   select NP, DESIGNATION, PRIXA FROM Produit where QTESTOCK<=50;
    MSG VARCHAR(50);
    QtApp NUMBER;
    lignec c%ROWTYPE;
        begin
    MSG:='';
    for lignec in c loop
    QtApp:= Calc_App(lignec.NP);
   MSG:='NP:'||lignec.NP ||'designation:'||lignec.designation||'Prix:'||lignec.PRIXA||'Quantite app:'||Calc_App(lignec.NP)|| 'prixtotal:' ||Calc_App(lignec.NP)*lignec.PRIXA;
  dbms_output.PUT_LINE(MSG);
   end loop;
   end;
      /
```     
