// Копіюй цей блок посилань у початок свого файлу
const IMGS = {
    THREAT: "https://r2.erweima.ai/i/9eE1pS-KTeSUnwMvY96lLA.png", // Червоний літак
    SAM_PICKUP: "https://r2.erweima.ai/i/2ZJ8s-WjS2m9mO7Gz5S_TQ.png", // Пікап ППО
    SAM_TRUCK: "https://r2.erweima.ai/i/6_rW9_D5S_6Z8uGfN_zR_A.png", // Вантажівка ППО
    SAM_TRACKED: "https://r2.erweima.ai/i/Y9_r8_D5S_6Z8uGfN_zR_A.png", // Гусенична ППО
    FACTORY_OK: "https://r2.erweima.ai/i/3S_r8_D5S_6Z8uGfN_zR_A.png", // Цілий завод
    FACTORY_DEAD: "https://r2.erweima.ai/i/1S_r8_D5S_6Z8uGfN_zR_A.png" // Зруйнований завод
};

// Клас ворога
class Threat {
    constructor(data) {
        this.currentLat = data.startLat;
        this.currentLng = data.startLng;
        this.hp = 100;
        
        this.marker = L.marker([this.currentLat, this.currentLng], {
            icon: L.icon({
                iconUrl: IMGS.THREAT,
                iconSize: [50, 50],
                iconAnchor: [25, 25]
            })
        }).addTo(map);
    }
    // ... (інша логіка руху)
}

// Клас ППО
class SAM {
    constructor(lat, lng, type) {
        this.lat = lat;
        this.lng = lng;
        
        // Вибираємо картинку залежно від типу
        let img = IMGS.SAM_PICKUP;
        if(type === 'heavy') img = IMGS.SAM_TRUCK;
        if(type === 'tracked') img = IMGS.SAM_TRACKED;

        this.marker = L.marker([lat, lng], {
            icon: L.icon({
                iconUrl: img,
                iconSize: [60, 40],
                iconAnchor: [30, 20]
            })
        }).addTo(map);
    }
}

// Функція для створення заводів (об'єктів захисту)
function createInfrastructure(lat, lng) {
    return L.marker([lat, lng], {
        icon: L.icon({
            iconUrl: IMGS.FACTORY_OK,
            iconSize: [70, 70],
            iconAnchor: [35, 35]
        })
    }).addTo(map);
}
