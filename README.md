# All plsql exercices are here ! 

<center>
  <div style="align : center;"> <img src="https://img.icons8.com/plasticine/452/oracle-pl-sql--v3.png" class="center"></div>
</center>

## Télecharger les instructions de création des tables ici : https://iset.uvt.tn/mod/resource/view.php?id=426236


## Résumé PLSQL :  https://iset.uvt.tn/mod/resource/view.php?id=454040

<hr>

### Correction de l'exercice numero 1  : ![Lien]:(https://ibb.co/r69ky1z)

1)
```sql
select count(*)
    from client
    ;
select ville, count(*)
    from client
   group by ville;
```

2)
```sql

select SUM(QteV*prixV)
    from vente, Produit
    where Vente.NP=Produit.NP;
```
3)

```sql
select SUM(QteV*prixV)
    from Vente natural join Produit
    GROUP BY NM;

2éme methode
select SUM(QteV*prixV) CA , NM
    from Vente join Produit on Vente.NP=Produit.NP
    GROUP BY NM;
```
4) 
```sql
select  NM , Nclt , SUM(QteV*prixV) CA
      from Vente, Produit
    where Vente.NP=Produit.NP
    and dateV between '01/01/2020' and '31/12/2020'
    group by  NM,Nclt;

2éme méthode :
select  NM , Nclt , SUM(QteV*prixV) CA
      from Vente, Produit
    where Vente.NP=Produit.NP
    and dateV > '01/01/2020' and  dateV <'31/12/2020'
    group by  NM,Nclt;
  
3éme methode :
select  NM , Nclt , SUM(QteV*prixV) CA
      from Vente, Produit
    where Vente.NP=Produit.NP
    and extract (year from dateV) = '2020'
    group by  NM,Nclt;
 
#4éme methode xD : 
select  NM , Nclt , SUM(QteV*prixV) CA
      from Vente, Produit
    where Vente.NP=Produit.NP
    and TO_CHAR(dateV,'yyyy')='2020'
    group by  NM,Nclt;
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
- Exercice 3 : 
> Ecrire une fonction __Cal_Chiff_Prod__ (IdClt In Numbre, Idp In Number) qui calcule le chiffre d’affaires d’un client donné pour un produit donné. 
Ce chiffre d’affaires est la somme des Qtev *Pu  du produit donné. 
> La Procédure développé sera appelée par un programme PL/SQL qui affiche  tous les clients de la  base . Pour chaque client, le programme doit afficher le chiffre d’affaire de chaque produit ainsi que le chiffre d’affaire globale de ce client.
> Le résultat retourné par le programme est des tuples (CodeCLt, IdArt, DesArt, ChiffAff).
```sql
create  OR REPLACE  function Calc_Chiff_Prod ( IDclt vente.Nclt%type , IDprod vente.np%type ) RETURN NUMBER IS 

CA NUMBER;

BEGIN 
select SUM(QteV*prixV) into CA
from vente  join produit on vente.np = produit.np
where vente.Nclt=IDclt and vente.np=IDprod;
return CA;
END;
/
```
<hr>

# Partie Trigger :
> 1/Créer  un déclencheur  qui affiche le numéro et le nom du magasin que l'on veut supprimer de la table Magasin : 
```sql
create or replace trigger TventeSupprime 
before delete 
on magasin 
for each row 

BEGIN
DBMS_output.put_line('suppression du magasin num ' || :OLD.NM || 'nom mag  ' || :old.GERANT  );
END; 
/
```
> 2/ Créer un trigger TventeSupprime qui permet de stocker toutes les ventes supprimée dans la table VenteSupprime , cette table est de même structure que la table Vente : 
```sql 
create or replace trigger TsuppVente
before delete on vente
for each row 

BEGIN
insert into vente_supp values( :old.NM , :old.np , :old.nclt , :old.DateV , :old.QteV , SYSDate );
END; 
/
```
> 3/ Ecrire un trigger qui utilise une séquence pour générer automatiquement  le numéro de  magasin  lors d’insertion d’un nouveau magasin : 
```sql
create or replace trigger insertautmagasin
before insert on magasin 
for each row 

BEGIN
select seqMag.nextval into :new.NM from Dual; 
END;
/
```
> 4/ Par mesure de sécurité on veut enregistrer dans une table TABLE_AUDIT  quel utilisateur a créé ou  a supprimé des lignes dans la table Magasin, avec la date de l'action, le type de l'action (ajout ou suppression) et le numéro de magasin concerné : 
- Voila la TABLE_AUDIT :  
```sql
Création de la table table_audit :
create table table_audit
(numero integer constraint  pk_audit primary key,
utilisateur varchar(30) not null,
date_modif date not null,
nm varchar (15) not null,
action varchar(15) not null);
```
- On creer le Trigger : 
```sql
create OR REPLACE trigger TmagSupp
after insert or delete 
on magasin
for each row 

BEGIN 

if inserting  then 
insert into table_audit values ( seqTableAudit.nextVal , user , sysDate, :new.NM, 'insertion');
end if;
if deleting then 
insert into table_audit values ( seqTableAudit.nextVal , user , sysDate, :old.NM, 'insertion');
end if;
END;
/
```
