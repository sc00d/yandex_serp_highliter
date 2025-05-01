<h1>Подсветка рекламы и своих сайтов в выдаче Яндекса на выдаче для ПК и для мобильных устройств</h1>


![image](https://github.com/user-attachments/assets/70c71f12-4a74-4fce-8cae-046eff35dd4d)

![image](https://github.com/user-attachments/assets/3873bf79-7b1a-45d3-a64f-387e05ad3719)

![image](https://github.com/user-attachments/assets/a30a4cdf-0f3c-436b-856f-ae702d5ffad4)



Решение для тех, кому надоело всматриваться в невидимые подписи "Реклама", (а теперь и значки ₽), которые притаились за хлебным крошками или где-то в углу. 
Со скриптом удобно смотреть где этот ваш "ТОП-10" органики находится в реальности у юзера и понимать почему CTR такой низкий =(

<h2>Как пользоваться?</h2>
Удобней всего использовать расширение https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld

Добавляем новое правило для URL Pattern: 
https://yandex.ru/search*, https://ya.ru/search*

Чтобы код заработал через расширение - надо разрешить "Запуск на старте" (внизу поля с JS кодом крайний правый значек)

![image](https://github.com/user-attachments/assets/87c6bc60-d1d5-45c1-b4fa-75435849f980)


Код вставляем в поле с JavaScript, сохраняем, наслаждаемся

— Работает в Моб и ПК выдаче
— Маркирует рекламу, которая появляется внутри органики при клике по одному из результатов

Чтобы пользоваться в инкогнито - просто разрешите расширению работать в нем (ПКМ по расширению - Управление расширениями - Разрешить использование в режиме инкогнито)

<h2>Об авторе</h2>

Автор: https://t.me/sc00d
Канал: https://t.me/seregaseo

***

<h2>Код</h2>

```
const myDomains = ['mydomain1.ru', 'mydomain2.com', 'mydomain3.pro'];

function markAds(node) {
  if (node.nodeType !== Node.ELEMENT_NODE) return;

  // Проверка релевантных элементов
  const isSearchResultItem = (
    node.matches('#search-result > div') ||
    node.matches('#search-result > li') ||
    node.matches('#search-result > li > div') ||
    (node.classList.contains('serp-item') && 
      (node.classList.contains('serp-item_card') || 
       node.classList.contains('serp-list__card'))) ||
    node.matches('[data-ajax-root="true"]') ||
    node.closest('[data-ajax-root="true"]')
  );

  if (isSearchResultItem) {
    let isAd = false;
    let hasMyDomain = false;

    // Проверка на рекламу по наличию ссылки с https://yandex.ru/search/
    const links = node.querySelectorAll('a');
    for (const link of links) {
      if (link.href.startsWith('https://yandex.ru/search/')) {
        isAd = true;
        break;
      }
    }

    // Проверка доменов, если не реклама
    if (!isAd) {
      for (const link of links) {
        try {
          const url = new URL(link.href, window.location.origin);
          const hostname = url.hostname.toLowerCase();
          if (myDomains.some((domain) => hostname === domain || hostname.endsWith('.' + domain))) {
            hasMyDomain = true;
            break;
          }
        } catch (e) {
          // Skip invalid URLs
        }
      }
    }

    // Раскрашивание элемента
    if (isAd) {
      node.style.backgroundColor = '#B79900';
    } else if (hasMyDomain) {
      node.style.backgroundColor = '#009159';
    }

    // Обработка вложенных PromoOffer
    const promoOffers = node.querySelectorAll('.PromoOffer');
    const color = isAd ? '#B79900' : hasMyDomain ? '#009159' : '';
    promoOffers.forEach((offer) => {
      offer.style.backgroundColor = color;
      offer.dataset.marked = 'processed';
    });

    // Пометка обработанного элемента
    node.dataset.marked = 'processed';
  }

  // Обработка корневых PromoOffer
  if (node.classList.contains('PromoOffer') && node.dataset.marked !== 'processed') {
    const parentNode = node.closest('#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"]');
    if (parentNode) {
      if (parentNode.style.backgroundColor === 'rgb(183, 153, 0)') {
        node.style.backgroundColor = '#B79900';
      } else if (parentNode.style.backgroundColor === 'rgb(0, 145, 89)') {
        node.style.backgroundColor = '#009159';
      }
      node.dataset.marked = 'processed';
    }
  }
}

// Наблюдение за #search-result
const targetNode = document.querySelector('#search-result') || document.body;
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    if (mutation.type === 'childList') {
      mutation.addedNodes.forEach((node) => {
        markAds(node);
        // Обработка вложенных элементов
        if (node.nodeType === Node.ELEMENT_NODE) {
          node.querySelectorAll('#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"], .PromoOffer')
            .forEach(markAds);
        }
      });
    }
  });
});

// Настройка наблюдения
const config = { childList: true, subtree: true };
observer.observe(targetNode, config);

// Начальная маркировка
const initialItems = targetNode.querySelectorAll('#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"], .PromoOffer');
initialItems.forEach(markAds);

// Периодическая проверка для AJAX-подгрузки
function checkNewElements() {
  const newItems = targetNode.querySelectorAll('#search-result > div, #search-result > li, #search-result > li > div, .serp-item.serp-item_card, .serp-item.serp-list__card, [data-ajax-root="true"], .PromoOffer');
  newItems.forEach((node) => {
    if (node.dataset.marked !== 'processed') {
      markAds(node);
    }
  });
}

// Обработка AJAX-подгрузки
let isChecking = false;
function handleAjaxContent() {
  if (isChecking) return;
  isChecking = true;
  let checkCount = 0;
  const maxChecks = 10; // 5 секунд
  const checkInterval = 500;

  const interval = setInterval(() => {
    checkNewElements();
    checkCount++;
    if (checkCount >= maxChecks) {
      clearInterval(interval);
      isChecking = false;
    }
  }, checkInterval);
}

// Обработка событий
document.addEventListener('click', (event) => {
  if (event.target.matches('a, button, [role="button"], .serp-item, .serp-item_card, .serp-list__card, [data-ajax-trigger], [data-load-more]')) {
    handleAjaxContent();
  }
});

document.addEventListener('scroll', () => {
  if (window.scrollY + window.innerHeight >= document.documentElement.scrollHeight - 100) {
    handleAjaxContent();
  }
});

// Перехват AJAX-запросов
(function () {
  if (window.jQuery) {
    const originalAjax = jQuery.ajax;
    jQuery.ajax = function () {
      const promise = originalAjax.apply(this, arguments);
      promise.done(() => setTimeout(handleAjaxContent, 100));
      return promise;
    };
  }

  const originalFetch = window.fetch;
  window.fetch = function () {
    const promise = originalFetch.apply(this, arguments);
    promise.then(() => setTimeout(handleAjaxContent, 100));
    return promise;
  };

  const originalOpen = XMLHttpRequest.prototype.open;
  XMLHttpRequest.prototype.open = function () {
    this.addEventListener('load', () => setTimeout(handleAjaxContent, 100));
    return originalOpen.apply(this, arguments);
  };
})();

// Функция для остановки наблюдения
function stopObserving() {
  observer.disconnect();
}



