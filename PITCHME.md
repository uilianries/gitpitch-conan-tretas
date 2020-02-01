## Resolvendo Tretas com Conan.io

#### A Ascenção e a Queda do Bárbaro

![conan](assets/img/conan-small.png)

---?image=assets/img/lego-dark-blue.png

#### Link desta Apresentação

http://bit.ly/3aL13xc

https://gitpitch.com/uilianries/gitpitch-conan-tretas

---?image=assets/img/lego-dark-green.png
@title[Sobre mim]

@div[left-70]
Olá!
<br>
<br>
**Uilian Ries**

<br>
<br>
Desenvolvedor C++ e Python
<br>
Trabalha para a **@jfrog**
<br>
<br>
Ex-Khomp (01/17 - 09/18)
<br>
<br>
@uilianries
<br>
@fa[github] @fa[twitter] @fa[linkedin]
@divend

@div[right-30]
![me](assets/img/me.png)
@divend

---?image=assets/img/lego-dark-blue.png

#### CONAN

@div[left-70]
<br>
<ul>
  <li>FOSS</li>
  <li>Licença MIT</li>
  <li>Descentralizado como GIT</li>
  <li>Manipula fontes e binários</li>
  <li>Geradores para CMake, Visual Studio, pkgconfig …</li>
  <li>Desenvolvido em Python</li>
  <li>+180 contribuidores</li>
  <li>Versão Atual 1.21.1 (Jan/2020)</li>
</ul>
@divend
@div[right-30]
![cpp](assets/img/logo.png)
@divend

---?image=assets/img/lego-dark-red.png

#### Sobre esta Apresentação

![crush-your-enemies](assets/gif/crush.gif)

- Apresentar novas features
- Discutir os sintomas do Dev
- Não haverá introdução ao Conan:
  - https://gitpitch.com/uilianries/gitpitch-conan
  - https://docs.conan.io/en/latest/videos.html

---?image=assets/img/lego-dark-red.png

#### Sobre esta Apresentação

- Deploy generator
- Deploying packages
- Hooks
- Python requires
- conandata.yml
- Editable packages
- Workflows
- Lockfiles
- Package Revisions
- Conan Center Index
- Conan Days

---?image=assets/img/lego-dark-green.png

#### Deploy Generator

    conan install . -g deploy

- Quando precisar instalar tudo em apenas um lugar
- Copiar todos os artefatos para o diretório atual
- Não vai para a cache do Conan
- Útil quando precisar empacotar para o cliente

---?image=assets/img/lego-dark-blue.png

#### Deploying Packages

```python
    def deploy(self)
        self.copy("foo", dst="bin", src="bin")
```

    conan create . user/testing

- Mesmo efeito que o gerador Deploy, porém
  já definido na receita.

---?image=assets/img/lego-dark-green.png

#### Hooks

https://github.com/conan-io/hooks

- Função Python que será executada em meio ao fluxo de trabalho
- Não modifica o cliente Conan ou as receitas
- Útil para executar passos customizados
- Exemplos: pylint na receita, disparar CI depois do upload

---?image=assets/img/lego-dark-green.png

#### Hooks

- Criar um script Python, utilizando os métodos suportados

```python
# ~/.conan/hooks/check_settings.py

def pre_export(output, conanfile, conanfile_path, reference, **kwargs):
    settings = getattr(conanfile, "settings")
    if settings and "cppstd" in settings:
        raise ConanException("The 'cppstd' setting is deprecated.")
```

---?image=assets/img/lego-dark-green.png

#### Hooks

- Métodos suportados (não todos):
  - pre_export
  - post_export
  - pre_source
  - post_source
  - pre_build
  - post_build
  - pre_package
  - post_package
  - pre_upload

---?image=assets/img/lego-dark-green.png

#### Hooks

Para ativar um hook:

    conan config set hooks.check_settings

Para desativar um hook:

    conan config rm hooks.check_settings

---?image=assets/img/lego-dark-blue.png

#### Python Requires

https://docs.conan.io/en/latest/extending/python_requires

- Permite compartilhar código e arquivos entre diferente receitas
- Útil para não precisar reescrever mesmo trecho
- Existe uma versão legado desta feature!

---?image=assets/img/lego-dark-blue.png

#### Python Requires

```python
# conanfile.py
from conans import ConanFile

myvar = 123

def myfunct():
    return 234

class Pkg(ConanFile):
    pass
```

    conan export . pyreq/0.1@user/channel

---?image=assets/img/lego-dark-blue.png

#### Python Requires

```python
from conans import ConanFile

class Pkg(ConanFile):
    python_requires = "pyreq/0.1@user/channel"

    def build(self):
        v = self.python_requires["pyreq"].module.myvar
        f = self.python_requires["pyreq"].module.myfunct()
        self.output.info("%s,%s" % (v, f))
```

---?image=assets/img/lego-dark-blue.png

#### Python Requires

Extendendo classes

```python
from conans import ConanFile

class MyBase(ConanFile):
    name = "base"
    version = "0.1.0"

    def source(self):
        self.output.info("no creo en las brujas")
    def build(self):
        self.output.info("pero que las hay las hay")
```

    conan export . user/channel

---?image=assets/img/lego-dark-blue.png

#### Python Requires

```python
from conans import ConanFile

class Pkg(ConanFile):
    name = "pkg"
    version = "0.1.0"
    python_requires = "base/0.1.0@uilianries/testing"
    python_requires_extend = "base.MyBase"

    def source(self):
        self.output.info("Miguel de Cervantes")
```

    conan create . pkg/0.1.0@user/channel

```
...
pkg/0.1@user/channel: Miguel de Cervantes
pkg/0.1@user/channel: pero que las hay las hay
```

---?image=assets/img/lego-dark-red.png

#### Conan Data (conandata.yml)

- Arquivo YAML carregado automaticamente pela receita
- Pode ser acessado através do atributo `conan_data`
- Útil quando uma mesma receita é usar para qualquer nova versão

---?image=assets/img/lego-dark-red.png

#### Conan Data (conandata.yml)

```yml
sources:
  1.0.0:
    url: "https://ftp.org/release/pkg-1.0.0.tar.gz"
    sha256: "430ae8354789de4fd19ee52f3b1f739e1f"
  1.0.1:
    url: "https://ftp.org/release/pkg-1.0.1.tar.gz"
    sha256: "d73a8da01e8bf8c7eda40b4c849150"
patches:
  1.0.0:
    patches: "0001-windows-build.patch"
  1.0.1:
    patches: []
```

---?image=assets/img/lego-dark-red.png

#### Conan Data (conandata.yml)

```python
class Foo(ConanFile):
    name = "pkg"

  def source(self):
      tools.get(**self.conan_data["sources"][self.version])
      for patch in self.conan_data["patches"][self.version]:
        tools.patch(**patch)
```

---?image=assets/img/lego-dark-green.png

#### Conan Center Index

https://github.com/conan-io/conan-center-index

- Lançado em 09/2019
- Centralização de todos os pacotes Conan
  - Absorção em maior parte do Bincrafters
  - Sobreposição de receitas mantidas por autores
  - Lembra o fluxo do Microsoft VCPKG
- Pacotes não mais portarão *namespace* em sua referência
  - e.g. zlib/1.2.11
  - Feature requer Conan >= 1.18.0

---?image=assets/img/lego-dark-green.png

#### Conan Center Index

- Base de pesquisa através do https://conan.io/center
- O interessado, abre um PR apenas com a receita e pronto
  - Anteriormente, era necessário cada um manter a integração
    contínua própria e submeter para Bintray cada pacote.
  - Contudo, existem regras de qualidade que limitam a
    liberdade de cada receita.

---?image=assets/img/lego-dark-green.png

#### Conan Days

![conan-days](assets/img/conan-days.png)

http://bit.ly/2TU94Ki

- Madri, 19/03 e 20/03, 2020
- Talks de empresas e suas experiências
- Treinamentos e feedback das empresas
- Ingresso variam entre €50 e €450

---?image=assets/img/lego-dark-red.png

#### Wheel of Pain

![wheel-of-pain](assets/gif/wheel.gif)

Vamos discutir os sintomas do Dev:

- Quais as maiores barreiras atuais?
- Como está sendo utilizado?
  - Receitas internas apenas?
  - Build from sources sempre?
- O que o Dev deseja da ferramenta?

---?image=assets/img/lego-dark-white.png

#### REFERÊNCIAS

* https://github.com/conan-io/conan
* https://github.com/bincrafters
* https://conan.io/center
* https://docs.conan.io
* https://github.com/conan-io/examples
* https://blog.conan.io

---?image=assets/img/lego-dark-white.png

### OBRIGADO!

##### Perguntas, Dúvidas, Curiosidades ?

Você pode me encontrar em:

**@uilianries** - twitter, github
cpplang.slack.com - canais #conan or #bincrafters
uilianries@gmail.com
https://conan.io
