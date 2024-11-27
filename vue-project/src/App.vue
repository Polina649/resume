<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import $ from 'jquery';

const file = ref(null);
const fileName = ref('');
const filePath = ref('');
const vacancies = ref([]); // Список вакансий
const code = ref('');
let addRecordRefResult = null;

function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function validatePhone(phone) {
  const phoneRegex = /^\+?[1-9]\d{1,14}$/; // Пример: +79991234567
  return phoneRegex.test(phone);
}


function uploadFileToServer(file, method = "PutFiileOnServerRef") {
  const reader = new FileReader();
  reader.onload = function (event) {
    const fileContent = event.target.result;
    const params = {
      Parameter01: btoa(fileContent),
      Parameter02: file.name, 
      Parameter03: 'Резюме',
      Parameter04: code.value, 
    };

    sendSoapRequest(method, params, (response) => {
      console.log("Файл успешно загружен на сервер:", response);
    });
  };

  if (file) {
    reader.readAsBinaryString(file); // Чтение файла как бинарной строки
  } else {
    console.error("Файл для загрузки не выбран.");
  }
}

// Обработчик изменения файла
function handleFileChange(event) {
  const selectedFile = event.target.files[0];
  if (selectedFile) {
    file.value = selectedFile;
    fileName.value = selectedFile.name;
    const actualPath = selectedFile.webkitRelativePath || `Файл прикреплен`;
    console.log = selectedFile.webkitRelativePath
    filePath.value = actualPath || `${selectedFile.name}`;
  }
}

// Открытие окна выбора файла
function triggerFileInput() {
  const fileInput = document.getElementById('resume');
  fileInput.click();
}

// Функция для создания SOAP-запроса
            // Функция для отправки SOAP-запроса
  function sendSoapRequest_3(method, params, outputElement, callback) {
        const soapBody = createSoapRequest(method, params);
        let parsedArray1 = null;
        let parsedArray2 = null;

        $.ajax({
            url: 'http://192.168.1.234/Test1C/ws/request.1cws',
            type: 'POST',
            data: soapBody,
            contentType: 'text/xml',
            headers: {
                Authorization: 'Basic ' + btoa("Admin:3432"),
                'SOAPAction': 'http://192.168.1.234/Test1C/request/' + method,
                'Accept': 'text/xml'
            },

    
    
    success: function(response) {
        const responseString = new XMLSerializer().serializeToString(response);
        const xmlDoc = new DOMParser().parseFromString(responseString, "text/xml");
        const namespace = "http://192.168.1.234/Test1C/request/";
        const returnElement = xmlDoc.getElementsByTagNameNS(namespace, "return")[0];

        if (returnElement) {
            const rawText = returnElement.textContent.trim();
            
            // Проверяем валидность JSON
            try {
                const jsonResponse = JSON.parse(rawText); // Пробуем распарсить как JSON
                parsedArray1 = jsonResponse;
                console.log("Parsed JSON response:", parsedArray1);

          // Извлечение данных из JSON
          document.getElementById("add_record_by_ref").addEventListener("click", function () {

        // Перебираем все элементы массива jsonResponse
        parsedArray2 = jsonResponse.map((item) => {
        const updatedItem = {};
        const key = Object.keys(item)[0];
        const currentValue = item[key];

        // Обновляем значение только если оно изменилось
        switch (key) {
            case "Телефон": {
                const newValue = document.getElementById("phone").value;
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            case "ЭлектроннаяПочта": {
                const newValue = document.getElementById('email').value;
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            case "Наименование": {
                const newValue = document.getElementById("full-name").value;
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            case "ОткликНаВакансию": {
                const newValue = document.getElementById("vacancy").value;
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            case "ИсточникРезюме": {
                const newValue = 'SamRegion.ru';
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            case "Комментарий": {
                const newValue = document.getElementById("comments").value;
                if (newValue !== currentValue) {
                    updatedItem[key] = newValue;
                }
                break;
            }

            default: {
                // Пропускаем ключи, которые не имеют значения
                break;
            }
        }

        return updatedItem;

            }).filter((item) => Object.keys(item).length > 0); // Удаляем пустые объекты
            // Выводим обновленный массив в консоль
            // console.log("Обновленный массив:", parsedArray2);
            const params = {
                Parameter01: 'Резюме',
                Parameter02: JSON.stringify(parsedArray2)
            };
            sendSoapRequest_AddRecordRef("AddRecordRef", params).then((soapResponse) => {


            }).catch((error) => {
                console.error("Ошибка при вызове AddRecordRef:", error);
                });
            });

        

            } catch (jsonError) {
                console.error("Ошибка парсинга JSON:", jsonError);
            }
            
                } else {
                    outputElement.text("Элемент <return> не найден в ответе.");
                }
            },

            error: function(xhr) {
                outputElement.text("Ошибка: " + xhr.status + " " + xhr.statusText);
            }
        });
    }

// Изменение sendSoapRequest_AddRecordRef
function sendSoapRequest_AddRecordRef(method, params) {
  const soapBody = createSoapRequest(method, params);

  return new Promise((resolve, reject) => {
    $.ajax({
      url: "http://192.168.1.234/Test1C/ws/request.1cws",
      type: "POST",
      data: soapBody,
      contentType: "text/xml",
      headers: {
        Authorization: "Basic " + btoa("Admin:3432"),
        SOAPAction: "http://192.168.1.234/Test1C/request/" + method,
        Accept: "text/xml",
      },
      success: function (response) {
        const serializer = new XMLSerializer();
        const responseString = serializer.serializeToString(response);

        const parser = new DOMParser();
        const xmlDoc = parser.parseFromString(responseString, "text/xml");
        // console.log(responseString);
        const namespace = "http://192.168.1.234/Test1C/request/";
        const returnElements = xmlDoc.getElementsByTagNameNS(namespace, "return");

        if (returnElements.length > 0) {
          const returnData = returnElements[0].textContent;
          code.value = returnData
          // console.log("Ответ сервера:", returnData);
          resolve(returnData);

          // Загружаем файл после успешного ответа
          if (file.value) {
            uploadFileToServer(file.value);
          }
        } else {
          console.error("Элемент <return> не найден в ответе.");
          reject("Элемент <return> не найден");
        }
      },
      error: function (err) {
        console.error("Ошибка AJAX:", err);
        reject(err);
      },
    });
  });
}



// Функция отправки SOAP-запроса
async function sendSoapRequest(method, params, callback) {
  const soapBody = createSoapRequest(method, params);

  try {
    const response = await axios.post('http://192.168.1.234/Test1C/ws/request.1cws', soapBody, {
      headers: {
        Authorization: 'Basic ' + btoa('Admin:3432'),
        'SOAPAction': `http://192.168.1.234/Test1C/request/${method}`,
        'Content-Type': 'text/xml',
        Accept: 'text/xml',
      },
    });

    const parser = new DOMParser();
    const xmlDoc = parser.parseFromString(response.data, 'text/xml');
    const namespace = 'http://192.168.1.234/Test1C/request/';
    const returnElement = xmlDoc.getElementsByTagNameNS(namespace, 'return')[0];

    if (returnElement) {
      const rawText = returnElement.textContent.trim();
      const cleanedText = rawText.replace(/[\[\]\{\}]/g, '').trim();
      const values = cleanedText.split(',').map(item => item.trim());
      if (callback) callback(values);
    }
  } catch (error) {
    console.error(`Ошибка: ${error.response?.status} ${error.response?.statusText}`);
  }
}

// Создание SOAP-запроса
function createSoapRequest(method, params) {
  const paramsXml = Object.entries(params)
    .map(([key, value]) => `<${key}>${value}</${key}>`)
    .join('');
  return `
    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:req="http://192.168.1.234/Test1C/request/">
      <soapenv:Header/>
      <soapenv:Body>
        <req:${method}>
          ${paramsXml}
        </req:${method}>
      </soapenv:Body>
    </soapenv:Envelope>
  `;
}

// Загружаем список вакансий при монтировании компонента
onMounted(() => {
  sendSoapRequest('GetListValuesOnFieldByRef', { Parameter01: 'Резюме', Parameter02: 'ОткликНаВакансию' }, (response) => {
    vacancies.value = response; // Обновляем список вакансий
  });
  sendSoapRequest_3('GetRecordByCode', { Parameter01: "Резюме", Parameter02: "000000004" }, (response) => {
     try {
       const parsedVacancies = JSON.parse(response);
       vacancies.value = parsedVacancies;
     } catch (error) {
       console.error('Ошибка при обработке списка вакансий:', error);
     }
  });

});




</script>

<template>
  <header>
    <div class="content_flex_text_h3">
      <button class="burger-button" aria-label="Меню">
        <span class="line"></span>
        <span class="line"></span>
        <span class="line"></span>
      </button>
      <div class="text_h3">Добавить резюме</div>
    </div>
  </header>

  <main>
    <div class="forms_containers">
      <div class="header_form_main">
        <h3 class="form_title">Заполните данные</h3>
      </div>
      
      <form class="form">
        <div class="content_form-group">
          <div class="form-group">
            <input type="text" id="full-name" name="full-name" required />
            <label for="full-name">Ф.И.О. <span class="required">*</span></label>
          </div>

          <div class="form-group">
            <select
              id="vacancy"
              name="vacancy"
              required
              ref="vacancySelect"
              @change="updateSelectTitle"
              :title="selectedTitle"
            >
              <option value="" disabled selected>Выберите вакансию</option>
              <option
                :title="vacancy"
                v-for="(vacancy, index) in vacancies"
                :key="index"
                :value="vacancy"
              >
                {{ vacancy }}
              </option>
            </select>


    <div class="tooltip" ref="tooltip">Подсказка</div>
          </div>
        </div>

        <div class="content_form-group">

          <div class="form-group">
          <input 
            type="email" 
            id="email" 
            name="email" 
            required 
            pattern="^[^@\\s]+@[^@\\s]+\.[^@\\s]+$" 
            title="Введите корректный адрес электронной почты, например example@mail.com"
          />
          <label for="email">Электронная почта <span class="required">*</span></label>
        </div>

          <div class="form-group">
          <input 
            type="text" 
            id="phone" 
            name="phone" 
            required 
            pattern="^\+?[1-9]\d{1,14}$" 
            title="Введите телефон в формате +71234567890 или 1234567890"
          />
          <label for="phone">Телефон <span class="required">*</span></label>
        </div>

      </div>

        <div class="form-group">
          <textarea id="comments" name="comments" rows="5"></textarea>
          <label for="comments">Комментарий</label>
        </div>

    <!-- Input file -->
    <div class="form-group_2">
      <!-- Кастомная кнопка для выбора файла -->
      <button type="button" class="custom-file-button" @click="triggerFileInput">
        <span v-if="fileName">{{ fileName }}</span>
        <span v-else>Прикрепить файл резюме</span>
      </button>

    <!-- Скрытый input для выбора файла -->
    <input
      type="file"
      id="resume"
      name="resume"
      ref="fileInput"
      @change="handleFileChange"
      style="display: none"
    />

    <p v-if="filePath" class="file-path">{{ filePath }}</p>

      </div>
        <button type="button" class="submit-button" id="add_record_by_ref">Отправить</button>
      </form>
    </div>
  </main>
</template>