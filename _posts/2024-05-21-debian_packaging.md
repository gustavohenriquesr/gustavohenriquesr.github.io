---
title: First experience with Debian packaging
date: 2024-05-21
categories: [University of São Paulo (USP), MAC5856 - Desenvolvimento de Software Livre]
tags: [software livre, open source]     # TAG names should always be lowercase
comments: false
---

Comments about my first experience with Debian packaging in MAC5856 - Desenvolvimento de Software Livre class.

Em continuação com o aprendizado em projetos open-source, iniciei pela primeira vez o processo de criação de pacotes para o sistema operacional [Debian](https://www.debian.org/). Para este feito, segui os dois tutoriais introdutórios de empacotamento de [Joenio](https://joenio.me/), um ativo contribuidor do Debian. Abaixo eu descrevo minha experiência na tentativa de seguir os tutoriais e os problemas que encontrei no processo.

Antes de inciar os tutoriais em si, eu precisei criar uma máquina virtual (VM) para criar um ambiente seguro e compatível à criação de pacotes. Para isso, utilizei a distribuição Debian de teste ("testing"), gerada a partir da distribuição "unstable" por um conjunto de scripts que buscam mover pacotes que provavelmente não possuem bugs críticos ao lançamento (release-critical). Com a criação da VM utilizando [virt-manager](https://virt-manager.org/), segui para a instalação de algumas dependências básicas para o desenvolvimento, além da configuração do git e cpan (para manipulação de pacotes e de arquivos perl). Assim, tudo ficou pronto para o inicio efetivo dos tutoriais.

### Tutorial 1

O [primeiro](https://joenio.me/tutorial-pacote-debian-parte1/#configurar-o-ambiente-de-desenvolvimento) tutorial descreve os passos para criação de pacotes da linguagem de programação [Perl](https://www.perl.org/) para o sistema operacional Debian. Todo o fluxo foi feito utilizando um pacote para teste, nomeado [Acme::Helloworld](https://metacpan.org/pod/Acme::Helloworld). Em primeiro lugar, criamos a versão incial do pacote a partir do download do código-fonte do upstream utilizando o comando `dh-make-perl`. Após o download, partimos para as correções básicas no template inicial do pacote. Nesse caso, realizei modificações simples nos arquivos `debian/copyright`, `debian/control` e `debian/changelog`, para atualizar as informações sobre copyright e licença, declaração de dependências, mantenedores, descrição, histórico de atualizações e versões de pacote. Fazendo as alterações apropriadas de acordo com as orientações do tutorial, fiz os commits devidos sem maiores problemas e, por fim, utilizei a ferramenta `cme` para verificar eventuais problemas, a qual não acusou nada significativo. Finalmente, parti para a construção (*build*) do pacote num *chroot* limpo (isto serve para criar um novo ambiente separado do diretório *root* do sistema principal, a fim de evitar vícios de ambiente e interferências de pacotes). Para isso, utilizei a ferramenta `pbuilder` e, sem nenhum problema, fui capaz de fazer a *build* do pacote. Após mais uma verificação para identificar eventuais problemas com [Lintian](https://wiki.debian.org/Lintian), enfim testei a instalação do pacote com `dpkg` e tudo ocorreu como esperado, finalizando o processo de criação do meu primeiro pacote Debian. 

### Tutorial 2

Neste segundo tutorial, a ideia foi seguir a partir do que foi feito no tutorial anterior para empacotar e submeter um pacote aos repositórios Debian. Para o seguimento do passo a passo, foi apresentado pacotes candidatos para seguir o tutorial, alguns que possuíam [bugs do tipo RFP](https://wnpp.debian.net/?type%5B%5D=RFP&project=perl&description=&owner%5B%5D=yes&owner%5B%5D=no&col%5B%5D=reporter&sort=dust%2Fasc) ou que eram bibliotecas Perl com [upload recente no cpan.org](https://metacpan.org/recent). Assim, decidi por escolher uma das bibliotecas Perl mais recente, a [Date::Holidays::NL](https://metacpan.org/pod/Date::Holidays::NL), que serve para verificar quais são os feriados oficiais dos Países Baixos. Com a biblioteca escolhida, segui para a construção do pacote seguindo praticamente os mesmos passos indicados no primeiro tutorial. Assim, alterei o arquivo `debian/copyright` com o nome do autor e seu e-mail como anteriormente, porém, a licença não estava indicada neste template inicial, que fui capaz de preencher corretamente após verificar o arquivo `LICENSE` do template incial (licença "*The (three-clause) BSD License*"). Já no arquivo `debian/control` a única alterações que foi preciso realizar foi a atualização da descrição do pacote, que fiz como segue abaixo. 

```patch
gustavo@debian:~/tests/part2/libdate-holidays-nl-perl/debian$ git diff HEAD~2
diff --git a/debian/control b/debian/control
index d31e1d1..16daddb 100644
--- a/debian/control
+++ b/debian/control
@@ -23,7 +23,8 @@ Depends: ${misc:Depends},
          libdate-holidays-abstract-perl,
          libdatetime-event-easter-perl,
          libdatetime-perl
-Description: Netherlands official holidays
- A Date::Holidays family member from the Netherlands
+Description: Perl module to find official holidays in Netherlands
+ The Date::Holidays::NL is a module with functions to find official holidays
+ in Netherlands
  .
- This description was automagically extracted from the module by dh-make-perl.
+ The module is a Date::Holidays family member from the Netherlands.
```

A partir daqui se iniciaram os problemas específicos da biblioteca que escolhi. Após a atualização da descrição, executei o comando `cme` no arquivo `debian/control` para correção de eventuais alertas e problemas relatados. o comando então devolveu alguns avisos de duas bibliotecas que estavam declaradas mas que não foram encontradas no repositório Debian: `libdate-holidays-abstract-perl` e `libdatetime-event-easter-perl`. A saída pode ser vista abaixo (o primeiro warning pode ser ignorado, como indicado no próprio tutorial):

```bash
gustavo@debian:~/tests/part2/libdate-holidays-nl-perl$ cme check dpkg-control debian/control
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Connecting to api.ftp-master.debian.org to check 3 package versions. Please wait...
Got info from api.ftp-master.debian.org for 1 packages.
Warning in 'source Standards-Version': Current standards version '4.7.0' is newer than lintian version (4.6.2). Please check your system
 (this cannot be fixed with 'cme fix' command)
Offending value: '4.7.0'
Connecting to api.ftp-master.debian.org to check libdate-holidays-abstract-perl versions. Please wait...
got no info for libdate-holidays-abstract-perl
Warning in 'source Build-Depends-Indep:0': package libdate-holidays-abstract-perl is unknown. Check for typos if not a virtual package.
Offending value: 'libdate-holidays-abstract-perl <!nocheck>'
Connecting to api.ftp-master.debian.org to check libdatetime-event-easter-perl versions. Please wait...
got no info for libdatetime-event-easter-perl
Warning in 'source Build-Depends-Indep:1': package libdatetime-event-easter-perl is unknown. Check for typos if not a virtual package.
Offending value: 'libdatetime-event-easter-perl <!nocheck>'
Warning in 'binary:"libdate-holidays-nl-perl" Depends:2': package libdate-holidays-abstract-perl is unknown. Check for typos if not a virtual package.
Offending value: 'libdate-holidays-abstract-perl'
Warning in 'binary:"libdate-holidays-nl-perl" Depends:3': package libdatetime-event-easter-perl is unknown. Check for typos if not a virtual package.
Offending value: 'libdatetime-event-easter-perl'debian/control
```

Considerando que eram "warnings" e não erros, tentei seguir o tutorial para criar um ambiente chroot isolado e criar o pacote. Porém, a execução do `pbuilder` falhou com o erro 
```bash 
...
E: pbuilder-satisfydepends failed.
...
```
justamente devido às bibliotecas não encontradas anteriormente.

A partir disso, decidi baixar manualmente as bibliotecas faltantes e construir cada um dos pacotes, como no primeiro tutorial. As bibliotecas referidas são [Date::Holidays::Abstract](https://metacpan.org/pod/Date::Holidays::Abstract) e [DateTime::Holidays::Easter](https://metacpan.org/pod/DateTime::Event::Easter). Em nenhum dos casos obtive êxito, pois estas não eram identificadas como pacotes Debian pelo comando `dh-make-perl`. Mesmo assim, decidi baixar diretamente das paǵinas das bibliotecas e instalá-las na VM. Da mesma forma, ao rodar o comando `cme` houveram os mesmo alertas relacionados à falta das bibliotecas, e novamente o `pbuilder` falhou. Depois dessas tentativas para instalação das dependências necessárias sem sucesso, decidi interromper o seguimento do segundo tutorial. 

### Considerações finais

Ao final, mesmo que não tenha completado integralmente a segunda parte, fui capaz de aprender o básico para construção de um pacote Debian (e ter pelo menos uma noção básica sobre submetê-los, embora não tenha feito efetivamente) a partir das orientações fornecidas nos dois tutoriais. Acredito que ainda é válida mais uma tentativa futuramente para tentar submeter o mesmo pacote (ou outro, com um problema mais simples) para fins de aprendizado. Os problemas que passei não pareceram tão grandes, apenas desafiadores o suficiente para que eu precise investir mais tempo do que tenho atualmente (considerando também o andamento da disciplina em questão).  
