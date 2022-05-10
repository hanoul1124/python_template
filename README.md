# Modern Python Dev Environment(in Local)
- 일관된 Python 개발환경을 제작하기 위한 베이스 템플릿
- Dependency, VirtualEnv, Code style, Test 등 Python Project에 필수적인 관리 요소 포함
- Local 개발환경 설정을 위한 세팅
- https://itnext.io/creating-a-modern-python-development-environment-3d383c944877 참조

## Base settings


1. `pyenv` 설치 및 shell python version 설정 (multiple Python version 다루기 위한 가상환경 설정)

    - `Python 3.6 +`



2. `Poetry` 설치 (Python Dependency management)

    - https://python-poetry.org/
    - https://blog.gyus.me/2020/introduce-poetry/



3. `EditorConfig` 설정 (IDE 환경에서 개발자 간 코딩스타일 일관성 부여)

  - Project의 Top level에 `.editorconfig` 설정 파일을 배치하여 사용

  - 해당 config file에 대해 built-in support를 제공하는 IDE: https://editorconfig.org/#pre-installed

  - 별도의 plugin을 설치해야 하는 IDE : https://editorconfig.org/#download



4. 프로젝트 시작(`poetry new PROJECT_NAME` & `git init`)

```sh
# 하위 디렉토리 구조를 확인하기 위한 Linux command.
# sudo apt install tree
# -a : hidden content를 함께 표시하는 옵션
> tree -a

# Base Setting이 완료된 프로젝트 디렉토리 구조
. # project root(poetry new "PROJECT NAME")
├── .editorconfig
├── .git
│   ├── ...
├── README.rst # 편의에 따라서 README.md로 교체
├── pyproject.toml
├── testproject
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_testproject.py
12 directories, 22 files
```



## Essential Packages


1. `pytest` 설치 (application test 자동화 프레임워크)

    - `Python 3.10 +` in  `pytest >= 6.2.5`
    - `Poetry`가 현재 `pytest` 설치 시 `5.xx` 버전을 설치하기 때문에, `pyproject.toml`에서 직접 설치할 `pytest` 버전을 지정해주도록 한다.

    - ```sh
        # --dev: development dependency로서 설치되도록 구분하는 옵션
        poetry add pytest pytest-cov --dev
        ```



2. `wemake-python-styleguide` 설치 (정적 코드 분석기인 flake8 기반의 strict Python linter)

    - `flake8` (Code style guide enforcement): https://flake8.pycqa.org/en/latest/

        - `lint`: 프로그래밍에서 의심스럽거나 에러를 발생하기 쉬운 코드에 표시(flag)를 달아 놓는 행위

    - https://wemake-python-stylegui.de/en/latest/#:~:text=strictest%20and%20most%20opinionated%20Python%20linter%20ever.

    -  ```sh
         poetry add wemake-python-styleguide --dev
         ```



3. `mypy` 설치 (static type checker)

    - ```bash
        poetry add mypy --dev
        ```



4. `safety` 설치 (Vulnerability check from project dependencies)

    - Vulnerabilty를 가진 Python 프로젝트의 87%는 vulnerable package를 업그레이드 하는 것으로 해결 가능하다.

    - `Safety`는 프로젝트의 사용된 dependency의 보안 취약점에 대해 검사한다.

    - ```bash
        poetry add safety --dev
        ```



5. `pre-commit` 설치 및 pre-commit config file 설정 (code commit 이전에 사전 설정된 검증 절차 실행. multi-language pre-commit hook framework)

    - 예를 들어, repository에 code를 commit하기 전 `pytest`를 자동 실행한다.

    - pre-commit 단계에서 실행할 검증 절차를 기록한 `.pre-commit-config.yaml` file을 프로젝트 내에 작성한다.

    - 본 템플릿에서는 다음과 같은 설정을 사용했다.

    - ```yaml
        # See https://pre-commit.com for more information
        # See https://pre-commit.com/hooks.html for more hooks
        repos:
          - repo: https://github.com/pre-commit/pre-commit-hooks
            rev: v3.2.0
            hooks:
              - id: check-added-large-files
              - id: trailing-whitespace
              - id: end-of-file-fixer
              - id: check-merge-conflict
              - id: check-case-conflict
              - id: check-json
              - id: check-toml
              - id: check-yaml
              - id: pretty-format-json
                args: [--autofix, --no-ensure-ascii, --no-sort-keys]
              - id: check-ast
              - id: debug-statements
              - id: check-docstring-first
          - repo: local
            hooks: # flake8, mypy, pytest, safety를 실행
              - id: flake8
                name: local flake8
                description: wemake-python-styleguide enforcement
                entry: poetry run flake8
                files: ^({PROJECT_NAME}/|tests/)
                args: ["--config=setup.cfg"]
                language: system
                types: [python]
              - id: mypy
                name: local mypy
                description: static type checker
                entry: poetry run mypy
                files: ^({PROJECT_NAME}/|tests/)
                language: system
                types: [python]
              - id: coverage
                name: local pytest coverage
                description: runs pytest along with coverage
                entry: poetry run pytest --cov {PROJECT_NAME} tests
                files: ^({PROJECT_NAME}/|tests/)
                language: system
                types: [python]
              - id: safety
                name: local safety
                description: check for vulnerabilities in packages.
                entry: poetry run safety check
                language: system
                types: [python]
                pass_filenames: false
        ```

    - ```bash
        poetry add pre-commit --dev
        poetry run pre-commit install

        # Run hooks once installed before commiting code.
        poetry run pre-commit run --all-files
        ```





6. `nitpick` 설치 (flake8 plugin for linting configuration files. 사전에 설정한 config를 정의하면 해당 설정이 포함되어 있는지 확인한다)

    - `.toml` 파일에 style 정의 후 `nitpick` 실행(`nitpick fix`, `nitpick check`)

    - ```toml
        # 본 템플릿에서는 pyproject.toml file에 다음과 같은 설정을 추가하였다.
        [tool.nitpick]
        style = "https://raw.githubusercontent.com/wemake-services/wemake-python-styleguide/master/styles/nitpick-style-wemake.toml"
        ```

    - ```bash
        poetry add nitpick --dev

        # 앞서 작성한 설정에 따라서, .gitignore, CHANGELOG.md 파일을 프로젝트 내에 포함시켜야 한다.
        touch .gitignore CHANGELOG.md

        poetry run nitpick fix # violation fix
        poetry run nitpick check

        # 위 command 결과 setup.cfg file이 생성(all the required configurations contained)
        ```

    - 사용 예시: https://github.com/Buzzvil/buzzvil-python-styleguide

s
