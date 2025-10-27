# Activer l’autocomplétion ``kubectl`` sur Windows PowerShell

## 1. Activer le module d’autocomplétion
Ouvre PowerShell en tant qu’administrateur et exécute :
```bash 
kubectl completion powershell | Out-String | Invoke-Expression
```
## 2 : La rendre permanente
Pour qu’elle soit chargée à chaque ouverture de PowerShell, ajoute la ligne suivante dans ton profil PowerShell :

```bash
kubectl completion powershell | Out-String | Invoke-Expression
```
Pour éditer ton profil :
```bash
notepad $PROFILE
```

Si ``$PROFILE`` n’existe pas :
```bash
New-Item -Type File -Force $PROFILE
```

## Troubleshooting
si l’exécution de scripts est désactivée sur ce système:
1. lister la politique actuelle
```bash
Get-ExecutionPolicy -List
```
2. Autoriser l’exécution de ton profil utilisateur uniquement
```bash
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
Puis confirme avec O (pour "Oui").
