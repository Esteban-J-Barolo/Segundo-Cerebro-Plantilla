---
created: 2026-02-02 16:34
status: Process
tags: 
  - type/inbox
origin: "[[Dashboard]]"
---
# 📥 Nueva Captura: Cosas que hice para config GIT
> [!ABSTRACT] 🧭 Panel de Control
> **Estado**: `INPUT[inlineSelect(option(Process), option(Review), option(OnHold), option(Dismissed)):status]`  

---
## 🖋️ Notas
### Resumen de tu carpeta `.ssh`
Debería verse así ahora:
- **`config`**: Las instrucciones de a qué Host corresponde cada llave.
- **`id_rsa_personal`**: Tu secreto (No lo toques).
- **`id_rsa_personal.pub`**: Lo que pegas en GitHub > Settings > SSH Keys.

```shell
PS C:\Users\esteb\Desktop\Segundo Cere> ssh-keygen -t rsa -b 4096 -f C:/Users/esteb/.ssh/id_rsa_personal
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): <--- ERROR, LE PUSE CONSTRSEA
```

config
```shell
# Cuenta Personal
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_personal
    IdentitiesOnly yes
```

### En la terminal como admin
#### Habilitar el servicio de SSH en Windows
```shell
PS C:\Users\esteb> Get-Service ssh-agent | Set-Service -StartupType Automatic
PS C:\Users\esteb> Start-Service ssh-agent
```

#### Añadir tu llave al agente
```shell
PS C:\Users\esteb> ssh-add "$env:USERPROFILE\.ssh\id_rsa_personal"
Enter passphrase for C:\Users\esteb\.ssh\id_rsa_personal:
Identity added: C:\Users\esteb\.ssh\id_rsa_personal (esteb@EstebanJ)
```

### El Clonador "Forzado"
```shell
git clone -c "core.sshCommand=ssh -i ~/.ssh/id_rsa_personal" git@github.com:Esteban-J-Barolo/Segundo-Cerebro.git
```

### Configuración post-clonación (Permanente)
```shell
cd Segundo-Cerebro
git config core.sshCommand "ssh -i ~/.ssh/id_rsa_personal"
```

### Quitarle la contraseña a la llave
```shell
ssh-keygen -p -f ~/.ssh/id_rsa_personal
```

## En el plugin Git de Obsidian
- activar el automático
- commit 30 min
- push 30 min
- mensaje commit: vault backup: {{date}} {{hostname}}
- hostnaem: Personal NR3
- pull in startup

---
## 🔗 Referencias
- 
---
`button-create-zettel` `button-delete-note`