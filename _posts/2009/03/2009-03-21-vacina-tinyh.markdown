---
layout: post
title:  "Vacina Tiny.H"
date:   2009-03-21
categories: malwares
tags: malware-analisys vacina
lang: br
ref: vacina-tiny-h
has-asciinema: false
---
A arte da engenharia reversa realmente é tão fantastica quanto ampla. Muitas pessoas só conhecem engenharia reversa como o método utilizado para criar cracks, keygens entre outras ferramentas do gênero. Porém, a engenharia reversa é utilizada amplamente para migração de código entre plataformas (geralmente drivers para hardware sem documentação), análise de vulnerabilidades em ferramentas de código fechado e para análise de malwares para criação de vacina.

Das criações de vacinas, pude fazer minha primeira vacina para um virus denominado TINY.H que estava disseminado nos laboratórios da faculdade. Ele se auto-copiava para dispositivos removíveis (pen-drivers), criando dois arquivos executáveis e um autorun, todos com permissões ocultas e de arquivos de sistema.

O problema é que, como o usuário disponível não tinha permissão para nada, não era permitido matar os processos do virus e, consequentemente, apagar os arquivos do pen-drive.

Para solucionar o problema, desenvolvi uma pseudo vacina em VBS que remove os processos e limpa o pen-driver, segue algumas rotinas necessárias. Basicamente foram utilizados dois objetos: o FileSystemObject e o Windows Management Instrumentation (WMI) como segue:

```vb
Set objWMIService = GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\cimv2")
Set objFSO = CreateObject("Scripting.FileSystemObject")
```

Tendo os objetos no escopo global, criei a seguinte rotina que remove as permissões de um dado arquivo:
```vb
'----------------------------------------------------------------------
' Esta função restaura as permissões dos arquivos de infecção para a
' NORMAL, pois os mesmo ficam com as permições SYSTEM, HIDDEN e ARCHIVE
'----------------------------------------------------------------------
Const FILE_ATTRIBUTE_NORMAL   = 128
 
sub RemovePermicoes(cArquivo)
    Wscript.Echo "  > " & cArquivo
    Set ObjFile = objFSO.GetFile(cArquivo)
    objFile.Attributes = FILE_ATTRIBUTE_NORMAL
end sub
```

Juntamente com ela, criei a seguinte função responsável pela remoção de um dado arquivo:
```vb
'----------------------------------------------------------------
' Esta função remove um arquivo ( no caso virótico ) passado como
' parametro
'----------------------------------------------------------------
sub RemoveArquivo(cArquivo)
	objFSO.DeleteFile(cArquivo)
	if objFSO.FileExists(cArquivo) then
	    Wscript.Echo "  > Arquivo [" & cArquivo & "] NÃO removido"
    else
	    Wscript.Echo "  > Arquivo [" & cArquivo & "] removido"
	end if
end sub
```

Antes de remover as permissões e apagar o ‘dito cujo’, foi necessário remover todos os processos com origem no arquivo, para isto escrevi a seguinte função:
```vb
'----------------------------------------------------------------------
' Esta função é responsável por remover os processos relacionados aos
' arquivos que identificam os virus
'----------------------------------------------------------------------
sub RemoveProcessos(cCaminho)
    Wscript.Echo "  > Caminho: " & cCaminho
    Set colProcessList = objWMIService.ExecQuery ("SELECT * FROM Win32_Process")
    For Each objProcess in colProcessList
        if objProcess.ExecutablePath = cCaminho then
            objProcess.Terminate()
            Wscript.Echo "    > PID: " & objProcess.ProcessId & " morto"
        end if
    next
end sub
```

Estas três funções podem ser reutilizadas em quaisquer vacinas que precise destas funcionalidades. Agora iremos partir para as tarefas específicas do virus TINY.H.

Primeiro precisamos identificar a presença do virus. Para isto peguei o nome dos arquivos que ele gera, chamados autorun.inf, explorer.exe e fooool.exe. Com isto, escrevi a seguinte função que verifica se um determinado dispositivo (H:, i:, etc) esta infectado:
```vb
'---------------------------------------------------------------------
' Verifica a possivel infecção em um dispositivo, tendo como parametro
' a letra do dispositivo e verificando através da existencia dos
' arquivos deixados pela infecção
'---------------------------------------------------------------------
function DispositivoInfectado(cDispositivo)
    DispositivoInfectado = objFSO.FileExists(cDispositivo + "\autorun.inf") AND _
                           objFSO.FileExists(cDispositivo + "\explorer.exe") AND _
                           objFSO.FileExists(cDispositivo + "\fooool.exe")
end function
```

Sabendo que um determinado dispositivo esta infectado, basta utilizar as rotinas já vistas para restaurar as permissões dos arquivos (remover o SYSTEM, HIDDEN e ARCHIVE), matar os respectivos processos e remover os arquivos:
```vb
'------------------------------------------------
' Função responsável pelo processo de desinfecção
'------------------------------------------------
function Desinfecta(cDispositivo)
    Wscript.Echo "----------------------------------"
    Wscript.Echo "======= Aplicando Recovery ======="
    Wscript.Echo "----------------------------------"
 
    Wscript.Echo "> Restaurando Permições para Original"
    RemovePermicoes cDispositivo & "\autorun.inf"
    RemovePermicoes cDispositivo & "\explorer.exe"
    RemovePermicoes cDispositivo & "\fooool.exe"
    Wscript.Echo ""
 
    Wscript.Echo "> Finalizando Processos Dependentes"    
    RemoveProcessos cDispositivo + "\explorer.exe"
    RemoveProcessos cDispositivo + "\fooool.exe"
    Wscript.Echo ""
 
    Wscript.Echo "> Apagando Arquivos"    
    RemoveArquivo cDispositivo + "\explorer.exe"
    RemoveArquivo cDispositivo + "\fooool.exe"
    Wscript.Echo ""
 
end function
```

E para nossa (pseudo-)vacina esta quase completa basta varrer todos os dispositivoes removíveis a procura de algum infectado:
```vb
'-------------------------------------------------------------
' Busca todos os dispositivos removiveis a procura da infeccao
'-------------------------------------------------------------
Set colDisks = objWMIService.ExecQuery("Select * from Win32_LogicalDisk Where DriveType = " & REMOVABLE_DRIVER & "")
 
 
Wscript.Echo "----------------------------------"
Wscript.Echo "====== Verificando Infecção ======"
Wscript.Echo "----------------------------------"
 
 
boolInfected = False
For Each objDisk in colDisks
 
    if objDisk.DeviceID  "A:" then ' Não vale disquete :P

        '----------------------------------------------
        ' Verifica a existencia da infeccao nos drivers
        '----------------------------------------------
        if DispositivoInfectado(objDisk.DeviceID) then
 
            Wscript.Echo "> Possível infecção TINY/H em (" + objDisk.DeviceID + ")"
            Wscript.Echo "  > " + objDisk.DeviceID + "\autorun.inf"
            Wscript.Echo "  > " + objDisk.DeviceID + "\explorer.exe"
            Wscript.Echo "  > " + objDisk.DeviceID + "\fooool.exe"
            Wscript.Echo ""
 
            Desinfecta objDisk.DeviceID
            boolInfected = True
        end if
    end if
Next
```

Bem. Isto foi só um básico de escrita de vacinas contra malwares. O código completo esta disponível para download <s>aqui</s>, porém o mais interessante seria disponibilizar minha análise (pra quem gosta de assembly). Ela esta um pouco bagunçada, portanto se tiver um tempinho extra irei organizar e posta a análise de meu primeira malware.

Maycon Maia Vitali ( 0ut0fBound )
