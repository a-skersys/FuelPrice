<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-fuel-secondary w-full text-center"
    >
      <p>
        Sie sehen sich gerade diese Stadt in der Vorschau an. Klicken Sie auf
        das <i class="fa-solid fa-circle-plus"></i> Symbol, um mit der
        Verfolgung dieser Stadt zu beginnen.
      </p>
    </div>
    <!-- Fuel Overview -->
    <div class="flex flex-col items-center text-white py-12 px-4">
      <h1 class="text-2xl mb-2">
        <i class="fa-solid fa-location-dot pr-1"></i> {{ route.params.city }}
      </h1>
      <h2 class="mb-2">
        <span class="capitalize">{{ route.query.type }}</span> in
        {{ route.query.range }} km Umkreis
      </h2>
      <ul class="text-sm mb-12">
        <li v-for="station in fuelData" :key="station.id">
          <div
            class="flex my-6 w-full bg-white rounded-lg shadow-md shadow-fuel-secondary"
            :class="[station.isOpen == false ? 'opacity-50' : 'opacity-100']"
          >
            <div class="h-24 w-1/6 my-6 mx-3 flex-shrink-0 overflow-hidden">
              <img
                v-if="Object.keys(Logos).includes(station.brand.split(' ')[0])"
                :src="brandLogo(station.brand.split(' ')[0])"
                :alt="brandAlt(station.brand.split(' ')[0])"
                class="h-full w-full p-2 object-contain object-center"
              />
              <img
                src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a1/Pictograms-nps-gas_station-2.svg/240px-Pictograms-nps-gas_station-2.svg.png"
                class="h-full w-full p-2 object-contain object-center opacity-20"
              />
            </div>
            <div class="m-6 ml-0 flex flex-1 flex-col">
              <div>
                <div
                  class="flex justify-between text-base font-medium text-gray-900"
                >
                  <h3>
                    <p>{{ station.brand }}</p>
                  </h3>
                </div>
                <p class="mt-1 text-sm text-gray-500">
                  {{ station.street }} {{ station.houseNumber }},
                  {{ station.postCode }} {{ station.place }}
                </p>
                <p class="mb-2 text-sm text-gray-500">{{ station.dist }} km</p>
              </div>
              <div class="flex flex-1 items-end justify-between text-sm">
                <p class="text-fuel-primary" v-if="station.isOpen == true">
                  GEÖFFNET
                </p>
                <p class="text-gray-400" v-if="station.isOpen == false">
                  GESCHLOSSEN
                </p>
              </div>
            </div>
            <div
              class="flex flex-col rounded-r-lg items-center justify-center bg-fuel-secondary w-1/4 md:w-1/3"
            >
              <p class="font-light text-white pb-2 text-2xl md:text-4xl">
                <span>{{ splitPrice(station.price.toString())[0] }}</span>
                <span>,</span>
                <span>{{ splitPrice(station.price.toString())[1] }}</span>
                <span
                  class="text-base align-top font-normal md:text-2xl md:align-text-top"
                  >{{ splitPrice(station.price.toString())[2] }}</span
                >
                <span class="font-light"> €</span>
              </p>
              <p
                v-if="
                  Number((station.price - averagePrice(fuelData)).toFixed(2)) *
                    100 <
                  0
                "
                class="text-fuel-primary"
              >
                {{
                  (
                    Math.abs(station.price - averagePrice(fuelData)) * 100
                  ).toFixed(0)
                }}
                ct billiger
              </p>
              <p
                v-else-if="
                  Number((station.price - averagePrice(fuelData)).toFixed(2)) *
                    100 >
                  0
                "
                class="text-fuel-wrong"
              >
                {{
                  (
                    Math.abs(station.price - averagePrice(fuelData)) * 100
                  ).toFixed(0)
                }}
                ct teurer
              </p>
              <p
                v-else-if="
                  Number((station.price - averagePrice(fuelData)).toFixed(2)) *
                    100 ==
                  0
                "
                class="text-gray-400 text-center"
              >
                durchschnitt-licher Preis
              </p>

              <p v-else class="text-black">
                {{ (station.price - averagePrice(fuelData)).toFixed(3) }}
              </p>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import { useRoute } from "vue-router";

const Logos: any = {
  ARAL: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/Aral_Logo.svg/1200px-Aral_Logo.svg.png",
    imageAlt: "Aral logo",
  },
  Shell: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/de/thumb/7/74/Royal_Dutch_Shell.svg/2194px-Royal_Dutch_Shell.svg.png",
    imageAlt: "Shell logo",
  },
  AVIA: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c0/AVIA_International_logo.svg/2560px-AVIA_International_logo.svg.png",
    imageAlt: "Avia logo",
  },
  OMV: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Omv_logo.svg/2560px-Omv_logo.svg.png",
    imageAlt: "OMV logo",
  },
  TotalEnergies: {
    imageSrc:
      "https://www.designtagebuch.de/wp-content/uploads/mediathek//2021/06/total-energies-logo.jpg",
    imageAlt: "TotalEnergies logo",
  },
  ESSO: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0e/Esso-Logo.svg/2560px-Esso-Logo.svg.png",
    imageAlt: "Esso logo",
  },
  bft: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/de/thumb/3/3f/Bft_logo.svg/1280px-Bft_logo.svg.png",
    imageAlt: "bft logo",
  },
  JET: {
    imageSrc: "https://upload.wikimedia.org/wikipedia/de/e/e5/JET.svg",
    imageAlt: "JET logo",
  },
  AGIP: {
    imageSrc:
      "https://upload.wikimedia.org/wikipedia/de/thumb/9/99/Agip_logo.svg/1200px-Agip_logo.svg.png",
    imageAlt: "AGIP logo",
  },
  HEM: {
    imageSrc:
      "data:image/jpeg;base64,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",
    imageAlt: "HEM logo",
  },
};

const route = useRoute();

const getFuelData = async () => {
  try {
    const result = await axios.get(
      `https://creativecommons.tankerkoenig.de/json/list.php?lat=${route.query.lat}&lng=${route.query.lng}&rad=${route.query.range}&sort=price&type=${route.query.type}&apikey=2ec47aaf-4e2f-1262-ab18-bfaec1e47136`
    );
    return result.data.stations;
  } catch (err) {
    console.log(err);
  }
  return;
};
const fuelData = await getFuelData();

function averagePrice(fuelData: any) {
  const priceArray = [];
  for (let i = 0; i < fuelData.length; i++) {
    priceArray[i] = fuelData[i].price;
  }
  const result =
    priceArray.reduce((a: number, b: number) => a + b, 0) / priceArray.length;
  return result;
}

function brandLogo(brand: string) {
  const imageSrc = Logos[brand].imageSrc;
  return imageSrc;
}

function brandAlt(brand: string) {
  const imageAlt = Logos[brand].imageAlt;
  return imageAlt;
}

const splitPrice = (price: String) => {
  const p1 = price.charAt(0);
  const p2 = price.charAt(2) + price.charAt(3);
  const p3 = price.charAt(4);
  const splitted = [p1, p2, p3];
  return splitted;
};
</script>
