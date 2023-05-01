---
layout: post
title: \{python\} Format code tự động sử dụng isort, black, flake8 và pre-commit
# subtitle: 
comments: true
cover-img: https://images.unsplash.com/photo-1534972195531-d756b9bfa9f2
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [python]
---

**TL; DR**: Trước khi thực hiện Git commit, mình sẽ dùng isort và black để format code tự động, sau đó dùng flake8 để kiểm tra lại lần nữa với chuẩn PEP8 (tất cả được cấu hình bằng pre-commit). Quá trình commit sẽ thành công nếu như không có lỗi xảy ra. Nếu xuất hiện lỗi, chúng ta sẽ quay lại sửa những chỗ cần thiết và commit lại lần nữa. Workflow này giúp giảm thời gian reformat code thủ công, nhờ đó chúng ta có thể tập trung hơn vào phần logic. Team làm việc với nhau cũng dễ dàng và hiệu quả hơn.

## Code Convention: Bạn có tốn quá nhiều công sức cho nó?

### PEP8

Code convention là một tập các quy tắc chuẩn khi lập trình. Nó bao gồm các quy ước đặt tên biến, tên function; cách thụt lề, cách comment; cách khai báo; hay những đề xuất về việc viết câu lệnh và tổ chức code, … Ngôn ngữ nào cũng có code convention riêng. Với Python, PEP8 (Python Enhancement Proposals) là một trong những bộ quy chuẩn được sử dụng rộng rãi nhất. Bạn có thể tìm hiểu về những quy tắc đó ở đây (Một điều khiến mình ngạc nhiên là nó đã có từ năm 2001 rồi mà gần 20 năm sau mình mới tìm hiểu về nó ☹️).

### Tại sao phải code đúng chuẩn?

Theo mình, giá trị lớn nhất của việc code đúng chuẩn chính là đảm bảo được sự nhất quán (consistency) trong mã nguồn. Điều này giúp code của bạn trở nên rõ ràng, dễ đọc, dễ hiểu cho chính bạn cũng như những người khác. Hãy tưởng tượng xem liệu bạn có:

ThÍcH ĐỌc mộT vĂn Bản mÀ NÓ đƯợc foRMat Như tHế nàY kHông?

Or 1 đoạn text # đc custom ntnày chẳghạn ?!??

(Mặc dù bạn gần như hiểu nó 100%? 🤔)

Đó là chưa kể những vòng loop lồng nhau với đủ loại khoảng cách và xuống hàng; những tên biến vô nghĩa như a, b, c, … xuất hiện dày đặc.

Suy cho cùng, viết code thực chất cũng giống như viết văn vậy. Chau chuốt câu chữ là một vấn đề rất đáng được lưu tâm.

### Tốn công sức với những điều không thực sự quan trọng

Đảm bảo được code đúng chuẩn khi làm việc nhóm khó hơn bạn nghĩ rất nhiều. Điều gì sẽ xảy ra nếu mỗi người đều code theo một style khác nhau? Okay, có thể bạn quy ước cả team theo chuẩn PEP8 từ đầu. Tuy nhiên, thỉnh thoảng khi review code, bạn sẽ gặp những comment đại loại như:

"Thêm một cái xuống dòng ở Line 23 dùm cái."

"Ký hiệu toán tử đặt trước tên biến thay vì cuối dòng trước đó nha cưng."

"Dòng code quá dài, chỉ nên giới hạn xy ký tự thôi nha. " (Điều này thuận tiện cho việc merge code. Khi mở hai cửa sổ cạnh nhau, ta có thể thấy được hết source code của hai versions mà không cần kéo qua kéo lại thanh trỏ ngang)

"Thêm khoảng trắng ở sau dấu hai chấm tại Line 47 nha."

"Xoá dấu cách dư sau dấu # ở dòng Line 117 giúp mình."

"import statement phải để trên đầu file nhé."

…

_(Ôi trông mình thật nhỏ mọn biết bao khi cứ suốt ngày đi vạch lưng người khác như thế! ☹️)_

Những vấn đề trên có thể họ biết, tuy nhiên không phải ai cũng nhớ và để ý hết tất - cả - những - gạch - đầu - dòng trong bộ quy tắc chuẩn. Và thế là, các thành viên trong team lại tốn thời gian và công sức review, feedback, sửa code, update với upstream, merge conflict nếu có, commit lại, rồi review lần 2… đôi khi chỉ với những cái nhỏ nhặt như vậy.

### Tập trung hơn vào những việc quan trọng nhờ vào một công cụ auto format ⭐

PEP8 chỉ là style guide, không phải là một tool hay library. Việc setup một công cụ format tự động theo chuẩn PEP8 là một vấn đề hết sức cần thiết, vì nó sẽ giúp bạn:

Không cần phải ghi nhớ những quy tắc dài dòng hay tốn thời gian review thủ công. Công cụ này sẽ xử lý hầu hết những vấn đề nhỏ đó cho bạn: báo cáo cho bạn biết những chỗ cần chỉnh sửa và có thể format mã nguồn một cách tự động.

Giảm thời gian thảo luận những thứ nhỏ nhặt, nhờ đó có thời gian tập trung vào những việc quan trọng hơn như viết logic xử lý hay tối ưu hoá performance.

Ít đi các trường hợp merge conflict vì lúc này code convention đã nhất quán giữa các thành viên từ newbie cho tới expert.

## Các packages cần thiết

Để có được một công cụ auto như thế chúng ta cần những "nguyên liệu" gì?

### isort

Package này sẽ sắp xếp tất cả biểu thức import trong Python file theo thứ tự bảng chữ cái và tự động phân chia thành các loại import khác nhau. Cài đặt chỉ cần chạy pip install isort.

Cùng xem xét ví dụ sau (được lấy từ trang chủ):

```python
# ===============

# myPythonFile.py

# ===============

from my_lib import Object
import os
from my_lib import Object3
from my_lib import Object2
import sys
from third_party import lib15, lib1, lib2, lib3, lib4, lib5, lib6, lib7, lib8, lib9, lib10, lib11, lib12, lib13, lib14
import sys

from __future__ import absolute_import

from third_party import lib3

print("Hey")
print("yo")
```

Kết quả sau khi chạy lệnh isort myPythonFile.py:

```python
from __future__ import absolute_import

import os
import sys

from third_party import (lib1, lib2, lib3, lib4, lib5, lib6, lib7, lib8,
                         lib9, lib10, lib11, lib12, lib13, lib14, lib15)

from my_lib import Object, Object2, Object3

print("Hey")
print("yo")
```

### Black

Black là một công cụ format code theo chuẩn của PEP8. Nó đi kèm với một vài lựa chọn khác về style mà nhóm phát triển tin rằng sẽ có ích trong việc tạo ra những dòng code dễ đọc và dễ bảo trì:

Số lượng ký tự trong một dòng là 88 thay vì 79 như PEP8.

Sử dụng nháy kép thay vì nháy đơn khi làm việc với chuỗi.

Nếu một hàm có nhiều tham số, mỗi tham số sẽ nằm trên một dòng.

Tất nhiên chúng ta có thể dễ dàng điều chỉnh những cấu hình mặc định bằng file pyproject.toml. Cài đặt bằng cách chạy dòng lệnh : `pip install black`

Cùng xem xét ví dụ sau:

```python
# ======================
# myAnotherPythonFile.py
# ======================

def calculate_something(a,b,c,d,e ,f ,g,h ):
    i = a+b
    j = (c+d) /2
    k =e*f
    m=  g-h
    #Check if i is an odd number
    if(i%2) ==1:
        i += 1
    return i + j+k+ m
```

Nếu thành thạo PEP8, bạn có thể thấy ngay ở đây có ít nhất 19 chỗ cần phải sửa 🤣. Chỉ với 9 dòng code người review phải feedback lại ít nhất 19 vấn đề về style. Dù nhận xét đúng nhưng chúng ta cũng chẳng vui vẻ gì khi nghe được.

Cùng chạy black myAnotherPythonFile.py và xem kết quả:

```
# ======================
# myAnotherPythonFile.py
# ======================

def calculate_something(a, b, c, d, e, f, g, h):
    i = a + b
    j = (c + d) / 2
    k = e * f
    m = g - h

    # Check if i is an odd number
    if (i % 2) == 1:
        i += 1
    return i + j + k + m
```

Voi-là! Mọi lỗi đã được fix xong chỉ trong một nốt nhạc.

### flake8

flake8 là một công cụ mạnh mẽ giúp kiểm tra code của bạn có thực sự tuân thủ chuẩn PEP8 hay không. Để black hoạt động tốt với flake8 (ngăn nó tạo ra nhiều lỗi và cảnh báo khác nhau), chúng ta cần liệt kê một số mã lỗi cần bỏ qua (mình sẽ chỉ rõ trong phần cấu hình bên dưới).

Chạy lệnh pip install flake8 để cài đặt. Sau đó test thử với ví dụ ở trên (lúc chưa format bằng black) bằng lệnh flake8 myAnotherPythonFile.py. Ngay lập tức, dưới console sẽ hiển thị các lỗi về typing mà mình đang mắc phải:

![Console sau khi chạy flake8](/assets/img/posts/flake8-console-log.jpg){: .mx-auto.d-block :}

### pre-commit

pre-commit là một framework để quản lý các pre-commit hooks trong Git. Ơ, mà Git Hook là gì ta?

Git Hook là những script đươc chạy tự động mỗi khi một sự kiện cụ thể nào đó diễn ra trong Git repository.

Trong trường hợp này, sự kiện ở đây chính là việc commit code. Chúng ta sẽ sử dụng pre-commit hook để kiểm tra những thay đổi trong code về convention lẫn style một cách tự động trước khi commit và tích hợp vào hệ thống. Nếu có điều gì bất thường, quá trình commit sẽ thất bại và chúng ta sẽ nhận được các thông báo lỗi liên quan để sửa. Quá trình commit chỉ thành công khi không có lỗi nào xảy ra.

Nãy giờ chúng ta mới chỉ giới thiệu và chạy thủ công các công cụ trên. Và giờ là lúc kết hợp chúng lại để chạy một cách tự động rồi!

## Workflow với pre-commit hooks

### Tổng quan

![Workflow với pre-commit (ảnh từ bài viết cuối bài)](/assets/img/posts/workflow-python-code-formatter.jpg){: .mx-auto.d-block :}

Trước khi thực hiện Git commit, mình sẽ dùng isort và black để format code tự động, sau đó dùng flake8 để kiểm tra lại lần nữa với chuẩn PEP8 (tất cả được cấu hình bằng pre-commit). Quá trình commit sẽ thành công nếu như không có lỗi xảy ra. Nếu xuất hiện lỗi, chúng ta sẽ quay lại sửa những chỗ cần thiết và commit lại lần nữa. Workflow này giúp giảm thời gian reformat code thủ công, nhờ đó tập trung hơn vào phần logic. Team làm việc với nhau cũng dễ dàng và hiệu quả hơn.

### Set-up step by step

(Đây là các file cấu hình mà mình đang sử dụng. Các bạn có thể tự điều chỉnh lại cho hợp với style hoặc nhu cầu của bản thân nhé!)

Tạo virtualvenv nếu cần và cài đặt pre-commit: pip3 install pre-commit Đừng quên thêm nó vào requirements.txt hoặc Pipfile sau khi cài đặt.

Cấu hình pre-commit bằng cách tạo file `.pre-commit-config.yaml` tại thư mục gốc có nội dung như sau (các bạn có thể điều chỉnh lên version mới nhất tại thời điểm đọc bài này):

```
repos:

- repo: <https://github.com/pre-commit/pre-commit-hooks>
    rev: v2.3.0
    hooks:
  - id: check-yaml
  - id: end-of-file-fixer
  - id: trailing-whitespace

- repo: <https://github.com/asottile/seed-isort-config>
    rev: v1.9.3
    hooks:
  - id: seed-isort-config

- repo: <https://github.com/pre-commit/mirrors-isort>
    rev: v4.3.21
    hooks:
  - id: isort

- repo: <https://github.com/psf/black>
    rev: 19.10b0
    hooks:
  - id: black
         language_version: python3.6

- repo: <https://gitlab.com/pycqa/flake8>
    rev: 3.8.3
    hooks:
  - id: flake8
```

Cấu hình isort bằng cách tạo file `.isort.cfg` tại thư mục gốc có nội dung như sau:

```
[settings]
line_length = 79
multi_line_output = 3
include_trailing_comma = True
```

Nếu các bạn để ý ở file cấu hình pre-commit, cùng với isort, chúng ta sử dụng thêm seed-isort-config để tự động thêm các packages vào known_third_party trong phần cấu hình isort (thay vì phải làm thủ công).

Cấu hình black bằng cách tạo file `pyproject.toml` tại thư mục gốc có nội dung như sau:

```
[tool.black]
line-length = 79
include = '\.pyi?$'
exclude = '''
/(
    \.git
    | \.hg
    | \.mypy_cache
    | \.tox
    | \.venv
    | _build
    | buck-out
    | build
    | dist
)/
```

Cấu hình flake8 bằng cách tạo file `.flake8` có nội dung như sau:

```
[flake8]
    ignore = E203, E266, E501, W503, F403, F401
    max-line-length = 79
    max-complexity = 18
    select = B,C,E,F,W,T4,B9
```

Sau khi cấu hình xong, chạy lệnh sau để hoàn tất việc cài đặt: `pre-commit install`

Cuối cùng, trước khi thực hiện Git commit, chạy lệnh sau: `pre-commit run --all-files`

Như các bạn có thể thấy, cài đặt workflow trên rất dễ dàng cho team vì mọi file cấu hình đều đã ở sẵn trong project, các thành viên chỉ cần chạy lệnh pre-commit install là xong.

### Về vấn đề CI/CD

Để đảm bảo hơn nữa thì việc kiểm tra code convention và style cũng nên được tích hợp vào CI (Continuous Integration) pipeline. Nên nhớ rằng vẫn có trường hợp ai đó trong team bỏ qua bước kiểm tra trong pre-commit bằng flag —no-verify khi commit. Tuy nhiên, phần setup này mình hiện tại chưa thực hiện. Mình sẽ update với các bạn ở một bài viết khác nhé!

## Một số phương thức thay thế

Nếu thấy phương pháp trên khá phức tạp và chưa cần thiết, bạn có thể chạy thủ công isort, blake, flake8 hoặc dùng các packages khác như autopep8 hoặc yapf. Ngoài ra, các IDE và Text Editor ngày nay đều hỗ trợ việc reformat code tự động. Nếu sử dụng Pycharm, bạn có thể thử Code > Reformat xem sao nhé!

## Lời kết

Những ưu điểm của workflow trên mình cũng đã nhắc đến vài lần trong bài viết này rồi. Các bạn có thể cân nhắc sử dụng nó theo các bước cài đặt mình đã giới thiệu ở trên. Cơ mà, thật sự thì, code style cũng chỉ là một đề xuất mà thôi. Bạn có thể làm bất cứ điều gì mà bạn muốn với codebase, miễn là đảm bảo được tính nhất quán, đặc biệt khi làm việc với những người khác. 🤗

Hẹn gặp lại các bạn ở những bài sau nhé! Happy coding!

## Reference

<https://ljvmiranda921.github.io/notebook/2018/06/21/precommits-using-black-and-flake8/>

<https://medium.com/staqu-dev-logs/keeping-python-code-clean-with-pre-commit-hooks-black-flake8-and-isort-cac8b01e0ea1>
