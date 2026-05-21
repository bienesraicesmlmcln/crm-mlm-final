const storageKey = "crm-leads-mlm-v3";
const legacyStorageKeys = ["crm-leads-mlm-v2", "crm-clientes-v1"];
const activityStorageKey = "crm-activity-mlm-v1";
const logoStorageKey = "crm-logo-v1";
const activeAdvisorStorageKey = "crm-active-advisor-v1";
const adminPasswordStorageKey = "crm-admin-password-v1";
const companyNameStorageKey = "crm-company-name-v1";
const themeStorageKey = "crm-theme-v1";
const staleDaysStorageKey = "crm-stale-days-v1";
const notificationStorageKey = "crm-notification-v1";
const advisorDirectoryStorageKey = "crm-advisors-v1";
const defaultAdminPassword = "AdminTamara";
const myAdvisorName = "Tamara Ortiz";
const appMode = document.body.dataset.mode || "advisor";

const stages = [
  "Nuevo Lead",
  "Interesado",
  "Buscando opciones",
  "Visita agendada",
  "Negociacion",
  "Apartado",
  "Cierre",
  "Perdido",
  "Seguimiento futuro"
];

const priorityRank = {
  Alta: 0,
  Media: 1,
  Baja: 2
};

const sampleClients = [
  {
    id: crypto.randomUUID(),
    createdAt: today(),
    name: "Ana Martinez",
    phone: "614 123 4567",
    type: "Venta",
    budget: 3200000,
    zone: "Canteras y Reliz",
    propertySent: "Casa Cumbres",
    source: "Instagram",
    priority: "Alta",
    lastContact: today(),
    nextAction: "Confirmar visita",
    followUpDate: addDays(1),
    stage: "Visita agendada",
    advisor: "Tamara Ortiz",
    clientStatus: "Activo",
    comments: "Busca casa con 3 recamaras, jardin y credito aprobado.",
    updatedAt: Date.now()
  },
  {
    id: crypto.randomUUID(),
    createdAt: addDays(-6),
    name: "Carlos Ruiz",
    phone: "614 987 1020",
    type: "Renta",
    budget: 18000,
    zone: "Distrito 1",
    propertySent: "Departamento ejecutivo",
    source: "Marketplace",
    priority: "Media",
    lastContact: addDays(-5),
    nextAction: "Esperar respuesta",
    followUpDate: addDays(-1),
    stage: "Buscando opciones",
    advisor: "Luis Herrera",
    clientStatus: "Frio",
    comments: "Quiere entrar el proximo mes. Prefiere amueblado.",
    updatedAt: Date.now() - 432000000
  },
  {
    id: crypto.randomUUID(),
    createdAt: addDays(-2),
    name: "Lucia Perez",
    phone: "614 555 8832",
    type: "Venta",
    budget: 2450000,
    zone: "Norte",
    propertySent: "",
    source: "Referido",
    priority: "Alta",
    lastContact: addDays(-2),
    nextAction: "Mandar opciones",
    followUpDate: today(),
    stage: "Nuevo Lead",
    advisor: "Tamara Ortiz",
    clientStatus: "Activo",
    comments: "Pidio opciones cerca de colegios. Responder rapido.",
    updatedAt: Date.now() - 172800000
  }
];

let clients = loadClients();
let activities = loadActivities();
let advisorDirectory = loadAdvisorDirectory();
let selectedId = clients[0]?.id ?? null;
let currentFilter = "all";
let currentStage = "all";
let currentAdvisor = appMode === "advisor" ? localStorage.getItem(activeAdvisorStorageKey) || "" : "all";

const elements = {
  list: document.querySelector("#clientList"),
  detail: document.querySelector("#detailPanel"),
  totalClients: document.querySelector("#totalClients"),
  pendingActions: document.querySelector("#pendingActions"),
  openValue: document.querySelector("#openValue"),
  closedClients: document.querySelector("#closedClients"),
  staleClients: document.querySelector("#staleClients"),
  clientCount: document.querySelector("#clientCount"),
  search: document.querySelector("#searchInput"),
  statusFilter: document.querySelector("#statusFilter"),
  advisorFilter: document.querySelector("#advisorFilter"),
  activeAdvisorName: document.querySelector("#activeAdvisorName"),
  changeAdvisor: document.querySelector("#changeAdvisor"),
  logoutAdmin: document.querySelector("#logoutAdmin"),
  advisorTable: document.querySelector("#advisorTable"),
  activityList: document.querySelector("#activityList"),
  dialog: document.querySelector("#clientDialog"),
  form: document.querySelector("#clientForm"),
  dialogTitle: document.querySelector("#dialogTitle"),
  deleteButton: document.querySelector("#deleteClient"),
  settingsButton: document.querySelector("#settingsButton"),
  settingsDialog: document.querySelector("#settingsDialog"),
  closeSettings: document.querySelector("#closeSettings"),
  changeLogoSetting: document.querySelector("#changeLogoSetting"),
  companyNameSetting: document.querySelector("#companyNameSetting"),
  themeSetting: document.querySelector("#themeSetting"),
  newAdvisorName: document.querySelector("#newAdvisorName"),
  createAdvisorButton: document.querySelector("#createAdvisorButton"),
  advisorSettingsList: document.querySelector("#advisorSettingsList"),
  newAdminPassword: document.querySelector("#newAdminPassword"),
  savePasswordButton: document.querySelector("#savePasswordButton"),
  staleDaysSetting: document.querySelector("#staleDaysSetting"),
  notificationSetting: document.querySelector("#notificationSetting"),
  settingsLogout: document.querySelector("#settingsLogout"),
  logoInput: document.querySelector("#logoInput"),
  brandLogo: document.querySelector("#brandLogo"),
  logoFrame: document.querySelector("#logoFrame")
};

setupAdminLogin();
setupAdvisorLogin();

document.querySelector("#newClientButton").addEventListener("click", () => openForm());
elements.settingsButton.addEventListener("click", openSettings);
elements.closeSettings.addEventListener("click", () => elements.settingsDialog.close());
elements.changeLogoSetting.addEventListener("click", () => elements.logoInput.click());
elements.companyNameSetting.addEventListener("change", saveCompanyName);
elements.themeSetting.addEventListener("change", saveTheme);
elements.createAdvisorButton.addEventListener("click", createAdvisor);
elements.savePasswordButton.addEventListener("click", saveAdminPassword);
elements.staleDaysSetting.addEventListener("change", saveFollowUpSettings);
elements.notificationSetting.addEventListener("change", saveFollowUpSettings);
elements.settingsLogout.addEventListener("click", logoutCurrentMode);
elements.logoInput.addEventListener("change", saveLogo);
document.querySelector("#closeDialog").addEventListener("click", closeForm);
document.querySelector("#cancelDialog").addEventListener("click", closeForm);
elements.form.addEventListener("submit", saveClient);
elements.deleteButton.addEventListener("click", deleteClient);
elements.search.addEventListener("input", render);
elements.statusFilter.addEventListener("change", () => {
  currentStage = elements.statusFilter.value;
  render();
});
if (elements.advisorFilter) {
  elements.advisorFilter.addEventListener("change", () => {
    currentAdvisor = elements.advisorFilter.value;
    render();
  });
}
if (elements.changeAdvisor) {
  elements.changeAdvisor.addEventListener("click", () => {
    localStorage.removeItem(activeAdvisorStorageKey);
    currentAdvisor = "";
    selectedId = null;
    lockAdvisorApp();
  });
}
if (elements.logoutAdmin) {
  elements.logoutAdmin.addEventListener("click", () => {
    sessionStorage.removeItem("mlm-admin-auth");
    lockAdminApp();
  });
}

document.querySelectorAll(".nav-item").forEach((button) => {
  button.addEventListener("click", () => {
    currentFilter = button.dataset.filter;
    document.querySelectorAll(".nav-item").forEach((item) => item.classList.remove("active"));
    button.classList.add("active");
    render();
  });
});

loadLogo();
loadCompanyName();
applyTheme(localStorage.getItem(themeStorageKey) || "mlm");
loadFollowUpSettings();
if (appMode === "advisor" && currentAdvisor) {
  unlockAdvisorApp();
  render();
}
if (appMode === "admin" && sessionStorage.getItem("mlm-admin-auth") === "ok") {
  unlockAdminApp();
  render();
}

function setupAdvisorLogin() {
  if (appMode !== "advisor") return;
  const form = document.querySelector("#advisorLoginForm");
  const input = document.querySelector("#advisorNameInput");
  if (currentAdvisor) {
    unlockAdvisorApp();
    return;
  }
  lockAdvisorApp();
  form.addEventListener("submit", (event) => {
    event.preventDefault();
    const advisorName = input.value.trim();
    if (!advisorName) return;
    currentAdvisor = advisorName;
    localStorage.setItem(activeAdvisorStorageKey, currentAdvisor);
    selectedId = clients.find((client) => client.advisor === currentAdvisor)?.id ?? null;
    unlockAdvisorApp();
    render();
  });
}

function unlockAdvisorApp() {
  const login = document.querySelector("#advisorLogin");
  const app = document.querySelector("#advisorApp");
  if (login) login.hidden = true;
  if (app) app.classList.remove("locked");
  if (elements.activeAdvisorName) elements.activeAdvisorName.textContent = currentAdvisor || "-";
}

function lockAdvisorApp() {
  const login = document.querySelector("#advisorLogin");
  const app = document.querySelector("#advisorApp");
  const input = document.querySelector("#advisorNameInput");
  if (login) login.hidden = false;
  if (app) app.classList.add("locked");
  if (input) input.value = "";
}

function setupAdminLogin() {
  if (appMode !== "admin") return;
  const login = document.querySelector("#adminLogin");
  const app = document.querySelector("#adminApp");
  const form = document.querySelector("#adminLoginForm");
  const password = document.querySelector("#adminPassword");
  const error = document.querySelector("#loginError");

  if (sessionStorage.getItem("mlm-admin-auth") === "ok") {
    login.hidden = true;
    app.classList.remove("locked");
    return;
  }

  app.classList.add("locked");
  form.addEventListener("submit", (event) => {
    event.preventDefault();
    if (password.value === getAdminPassword()) {
      sessionStorage.setItem("mlm-admin-auth", "ok");
      login.hidden = true;
      app.classList.remove("locked");
      render();
      return;
    }
    error.hidden = false;
  });
}

function unlockAdminApp() {
  const app = document.querySelector("#adminApp");
  const login = document.querySelector("#adminLogin");
  if (app) app.classList.remove("locked");
  if (login && sessionStorage.getItem("mlm-admin-auth") === "ok") login.hidden = true;
}

function lockAdminApp() {
  const app = document.querySelector("#adminApp");
  const login = document.querySelector("#adminLogin");
  const password = document.querySelector("#adminPassword");
  const error = document.querySelector("#loginError");
  if (app) app.classList.add("locked");
  if (login) login.hidden = false;
  if (password) password.value = "";
  if (error) error.hidden = true;
}

function getAdminPassword() {
  return localStorage.getItem(adminPasswordStorageKey) || defaultAdminPassword;
}

function today() {
  return new Date().toISOString().slice(0, 10);
}

function addDays(days) {
  const date = new Date();
  date.setDate(date.getDate() + days);
  return date.toISOString().slice(0, 10);
}

function loadClients() {
  const saved = localStorage.getItem(storageKey) || legacyStorageKeys.map((key) => localStorage.getItem(key)).find(Boolean);
  if (!saved) {
    localStorage.setItem(storageKey, JSON.stringify(sampleClients));
    return sampleClients;
  }

  try {
    const parsed = JSON.parse(saved).map(normalizeClient);
    localStorage.setItem(storageKey, JSON.stringify(parsed));
    return parsed;
  } catch {
    return sampleClients;
  }
}

function normalizeClient(client) {
  const lastContact = client.lastContact || client.lastMovement || dateFromTimestamp(client.updatedAt) || today();
  const stage = client.stage || stageFromOldStatus(client.status);
  return {
    id: client.id || crypto.randomUUID(),
    createdAt: client.createdAt || dateFromTimestamp(client.createdAt) || lastContact,
    name: client.name || "",
    phone: client.phone || "",
    type: client.type === "Compra" ? "Venta" : client.type || "Venta",
    budget: Number(client.budget ?? client.value ?? 0),
    zone: client.zone || client.company || "",
    propertySent: client.propertySent || "",
    source: client.source || "WhatsApp",
    priority: client.priority || "Media",
    lastContact,
    nextAction: client.nextAction || "",
    followUpDate: client.followUpDate || client.nextDate || "",
    stage,
    advisor: client.advisor || myAdvisorName,
    clientStatus: client.clientStatus || clientStatusFromStage(stage),
    comments: client.comments || client.notes || "",
    updatedAt: Number(client.updatedAt || Date.now())
  };
}

function dateFromTimestamp(value) {
  if (!value) return "";
  if (typeof value === "string" && /^\d{4}-\d{2}-\d{2}$/.test(value)) return value;
  const date = new Date(value);
  return Number.isNaN(date.getTime()) ? "" : date.toISOString().slice(0, 10);
}

function stageFromOldStatus(status) {
  if (status === "Nuevo") return "Nuevo Lead";
  if (status === "Propuesta enviada") return "Negociacion";
  if (status === "Cerrado") return "Cierre";
  if (status === "Perdido") return "Perdido";
  return status || "Nuevo Lead";
}

function clientStatusFromStage(stage) {
  if (stage === "Cierre") return "Cerrado";
  if (stage === "Perdido") return "Perdido";
  if (stage === "Seguimiento futuro") return "Seguimiento futuro";
  return "Activo";
}

function loadActivities() {
  const saved = localStorage.getItem(activityStorageKey);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch {
      return [];
    }
  }

  const initial = [
    activityItem("Tamara Ortiz", "actualizo cliente", "Ana Martinez"),
    activityItem("Tamara Ortiz", "agendo visita", "Ana Martinez"),
    activityItem("Luis Herrera", "registro seguimiento", "Carlos Ruiz")
  ];
  localStorage.setItem(activityStorageKey, JSON.stringify(initial));
  return initial;
}

function loadAdvisorDirectory() {
  const saved = localStorage.getItem(advisorDirectoryStorageKey);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch {
      return [];
    }
  }
  return [
    { name: "Tamara Ortiz", active: true },
    { name: "Luis Herrera", active: true }
  ];
}

function persistAdvisorDirectory() {
  localStorage.setItem(advisorDirectoryStorageKey, JSON.stringify(advisorDirectory));
}

function activityItem(advisor, action, clientName) {
  return {
    id: crypto.randomUUID(),
    advisor,
    action,
    clientName,
    createdAt: Date.now()
  };
}

function persist() {
  localStorage.setItem(storageKey, JSON.stringify(clients));
}

function persistActivities() {
  localStorage.setItem(activityStorageKey, JSON.stringify(activities.slice(0, 20)));
}

function recordActivity(advisor, action, clientName) {
  activities.unshift(activityItem(advisor || myAdvisorName, action, clientName));
  activities = activities.slice(0, 20);
  persistActivities();
}

function loadLogo() {
  const savedLogo = localStorage.getItem(logoStorageKey);
  if (!savedLogo) {
    elements.logoFrame.classList.add("has-logo");
    return;
  }
  elements.brandLogo.src = savedLogo;
  elements.logoFrame.classList.add("has-logo");
}

function saveLogo(event) {
  const file = event.target.files?.[0];
  if (!file) return;

  const reader = new FileReader();
  reader.addEventListener("load", () => {
    localStorage.setItem(logoStorageKey, reader.result);
    elements.brandLogo.src = reader.result;
    elements.logoFrame.classList.add("has-logo");
  });
  reader.readAsDataURL(file);
}

function loadCompanyName() {
  const name = localStorage.getItem(companyNameStorageKey) || "MLM Bienes Raices";
  if (elements.companyNameSetting) elements.companyNameSetting.value = name;
  document.querySelectorAll(".logo-frame img").forEach((img) => {
    img.alt = name;
  });
}

function saveCompanyName() {
  const name = elements.companyNameSetting.value.trim() || "MLM Bienes Raices";
  localStorage.setItem(companyNameStorageKey, name);
  loadCompanyName();
}

function applyTheme(theme) {
  document.body.dataset.theme = theme;
  if (elements.themeSetting) elements.themeSetting.value = theme;
}

function saveTheme() {
  localStorage.setItem(themeStorageKey, elements.themeSetting.value);
  applyTheme(elements.themeSetting.value);
}

function loadFollowUpSettings() {
  if (elements.staleDaysSetting) elements.staleDaysSetting.value = localStorage.getItem(staleDaysStorageKey) || "3";
  if (elements.notificationSetting) elements.notificationSetting.value = localStorage.getItem(notificationStorageKey) || "Activadas";
}

function saveFollowUpSettings() {
  localStorage.setItem(staleDaysStorageKey, elements.staleDaysSetting.value || "3");
  localStorage.setItem(notificationStorageKey, elements.notificationSetting.value);
  render();
}

function openSettings() {
  loadCompanyName();
  applyTheme(localStorage.getItem(themeStorageKey) || "mlm");
  loadFollowUpSettings();
  renderAdvisorSettings();
  elements.settingsDialog.showModal();
}

function createAdvisor() {
  const name = elements.newAdvisorName.value.trim();
  if (!name) return;
  if (!advisorDirectory.some((advisor) => advisor.name.toLowerCase() === name.toLowerCase())) {
    advisorDirectory.push({ name, active: true });
    advisorDirectory.sort((a, b) => a.name.localeCompare(b.name));
    persistAdvisorDirectory();
  }
  elements.newAdvisorName.value = "";
  renderAdvisorSettings();
  render();
}

function renderAdvisorSettings() {
  if (!elements.advisorSettingsList) return;
  const advisors = advisorOptions(true);
  elements.advisorSettingsList.innerHTML = advisors
    .map((advisor) => {
      const record = advisorDirectory.find((item) => item.name === advisor) || { name: advisor, active: true };
      return `
        <div class="advisor-setting-row">
          <input class="advisor-edit-input" data-advisor="${escapeHtml(advisor)}" value="${escapeHtml(advisor)}" />
          <button type="button" class="secondary-button advisor-save" data-advisor="${escapeHtml(advisor)}">Editar</button>
          <button type="button" class="ghost-button advisor-toggle" data-advisor="${escapeHtml(advisor)}">${record.active === false ? "Activar" : "Desactivar"}</button>
        </div>`;
    })
    .join("");
  elements.advisorSettingsList.querySelectorAll(".advisor-save").forEach((button) => {
    button.addEventListener("click", () => renameAdvisor(button.dataset.advisor));
  });
  elements.advisorSettingsList.querySelectorAll(".advisor-toggle").forEach((button) => {
    button.addEventListener("click", () => toggleAdvisor(button.dataset.advisor));
  });
}

function renameAdvisor(oldName) {
  const input = [...elements.advisorSettingsList.querySelectorAll(".advisor-edit-input")].find((item) => item.dataset.advisor === oldName);
  const newName = input?.value.trim();
  if (!newName || newName === oldName) return;
  clients.forEach((client) => {
    if (client.advisor === oldName) client.advisor = newName;
  });
  const record = advisorDirectory.find((advisor) => advisor.name === oldName);
  if (record) record.name = newName;
  if (currentAdvisor === oldName) {
    currentAdvisor = newName;
    localStorage.setItem(activeAdvisorStorageKey, newName);
  }
  persist();
  persistAdvisorDirectory();
  renderAdvisorSettings();
  render();
}

function toggleAdvisor(name) {
  let record = advisorDirectory.find((advisor) => advisor.name === name);
  if (!record) {
    record = { name, active: true };
    advisorDirectory.push(record);
  }
  record.active = record.active === false;
  persistAdvisorDirectory();
  renderAdvisorSettings();
  render();
}

function saveAdminPassword() {
  const value = elements.newAdminPassword.value.trim();
  if (!value) return;
  localStorage.setItem(adminPasswordStorageKey, value);
  elements.newAdminPassword.value = "";
}

function logoutCurrentMode() {
  elements.settingsDialog.close();
  if (appMode === "admin") {
    sessionStorage.removeItem("mlm-admin-auth");
    lockAdminApp();
    return;
  }
  localStorage.removeItem(activeAdvisorStorageKey);
  currentAdvisor = "";
  selectedId = null;
  lockAdvisorApp();
}

function render() {
  const filtered = getFilteredClients();
  if (!filtered.some((client) => client.id === selectedId)) {
    selectedId = filtered[0]?.id ?? null;
  }
  renderAdvisorFilter();
  renderMetrics();
  renderAdvisorTable();
  renderActivity();
  renderList(filtered);
  renderDetail();
}

function renderAdvisorFilter() {
  const advisors = advisorOptions(appMode === "admin");

  if (elements.activeAdvisorName) {
    elements.activeAdvisorName.textContent = currentAdvisor || "-";
  }

  if (!elements.advisorFilter) return;
  const options = ['<option value="all">Todos</option>', ...advisors.map((advisor) => `<option>${escapeHtml(advisor)}</option>`)];
  if (currentAdvisor !== "all" && !advisors.includes(currentAdvisor)) currentAdvisor = "all";
  elements.advisorFilter.innerHTML = options.join("");
  elements.advisorFilter.value = currentAdvisor;
}

function getFilteredClients() {
  const query = elements.search.value.trim().toLowerCase();
  return clients
    .filter((client) => currentStage === "all" || client.stage === currentStage)
    .filter((client) => {
      if (appMode === "advisor") return client.advisor === currentAdvisor;
      return currentAdvisor === "all" || client.advisor === currentAdvisor;
    })
    .filter(matchesSection)
    .filter((client) => {
      const haystack = [
        client.name,
        client.phone,
        client.type,
        client.zone,
        client.propertySent,
        client.source,
        client.priority,
        client.nextAction,
        client.stage,
        client.advisor,
        client.clientStatus,
        client.comments
      ]
        .join(" ")
        .toLowerCase();
      return haystack.includes(query);
    })
    .sort((a, b) => {
      const aStale = isStale(a) ? 0 : 1;
      const bStale = isStale(b) ? 0 : 1;
      const dateA = new Date(a.followUpDate || "2999-12-31");
      const dateB = new Date(b.followUpDate || "2999-12-31");
      return aStale - bStale || dateA - dateB || priorityRank[a.priority] - priorityRank[b.priority];
    });
}

function matchesSection(client) {
  if (currentFilter === "all") return true;
  if (currentFilter === "Mis clientes") return appMode === "advisor" ? client.advisor === currentAdvisor : client.advisor === myAdvisorName;
  if (currentFilter === "Nuevos") return client.stage === "Nuevo Lead";
  if (currentFilter === "Cerrados") return client.stage === "Cierre";
  if (currentFilter === "Sin seguimiento") return isStale(client);
  if (currentFilter === "Propuestas") return Boolean(client.propertySent) || ["Negociacion", "Apartado"].includes(client.stage);
  if (currentFilter === "En seguimiento") {
    return !["Nuevo Lead", "Cierre", "Perdido"].includes(client.stage) && !isStale(client);
  }
  return true;
}

function isStale(client) {
  if (["Cierre", "Perdido"].includes(client.stage)) return false;
  if (!client.nextAction || !client.followUpDate) return true;
  if (daysSince(client.lastContact) > Number(localStorage.getItem(staleDaysStorageKey) || 3)) return true;
  return dateIsPast(client.followUpDate);
}

function daysSince(dateValue) {
  if (!dateValue) return 999;
  const start = new Date(`${dateValue}T12:00:00`);
  const now = new Date(`${today()}T12:00:00`);
  return Math.floor((now - start) / 86400000);
}

function dateIsPast(dateValue) {
  if (!dateValue) return true;
  return new Date(`${dateValue}T12:00:00`) < new Date(`${today()}T12:00:00`);
}

function renderMetrics() {
  const metricClients = appMode === "advisor" ? clients.filter((client) => client.advisor === currentAdvisor) : clients;
  const openClients = metricClients.filter((client) => !["Cierre", "Perdido"].includes(client.stage));
  elements.totalClients.textContent = metricClients.length;
  elements.pendingActions.textContent = openClients.filter((client) => client.nextAction && client.followUpDate).length;
  elements.openValue.textContent = money(openClients.reduce((sum, client) => sum + Number(client.budget || 0), 0));
  elements.closedClients.textContent = metricClients.filter((client) => client.stage === "Cierre").length;
  elements.staleClients.textContent = metricClients.filter(isStale).length;
}

function renderAdvisorTable() {
  if (!elements.advisorTable) return;
  const advisors = advisorOptions(true);
  elements.advisorTable.innerHTML = `
    <div class="advisor-row advisor-head">
      <span>Asesor</span><span>Leads</span><span>Seguim.</span><span>Visitas</span><span>Propuestas</span><span>Cierres</span><span>Sin seg.</span>
    </div>
    ${advisors
      .map((advisor) => {
        const owned = clients.filter((client) => (client.advisor || "Sin asesor") === advisor);
        return `
          <div class="advisor-row">
            <strong>${escapeHtml(advisor)}</strong>
            <span>${owned.length}</span>
            <span>${owned.filter((client) => !["Nuevo Lead", "Cierre", "Perdido"].includes(client.stage)).length}</span>
            <span>${owned.filter((client) => client.stage === "Visita agendada").length}</span>
            <span>${owned.filter((client) => Boolean(client.propertySent) || ["Negociacion", "Apartado"].includes(client.stage)).length}</span>
            <span>${owned.filter((client) => client.stage === "Cierre").length}</span>
            <span>${owned.filter(isStale).length}</span>
          </div>`;
      })
      .join("")}
  `;
}

function renderActivity() {
  if (!elements.activityList) return;
  elements.activityList.innerHTML = activities
    .slice(0, 6)
    .map(
      (activity) => `
        <div class="activity-item">
          <span></span>
          <p><strong>${escapeHtml(activity.advisor)}</strong> ${escapeHtml(activity.action)}: ${escapeHtml(activity.clientName)}</p>
        </div>`
    )
    .join("");
}

function renderList(filtered) {
  elements.clientCount.textContent = `${filtered.length} ${filtered.length === 1 ? "resultado" : "resultados"}`;

  if (!filtered.length) {
    elements.list.innerHTML = `<p class="empty-state">No hay leads con esos filtros.</p>`;
    return;
  }

  elements.list.innerHTML = filtered
    .map((client) => {
      const stale = isStale(client);
      const advisors = advisorOptions();
      return `
        <article class="client-card tone-${toneClass(client)} ${client.id === selectedId ? "selected" : ""}" data-id="${client.id}">
          <button class="card-select" data-id="${client.id}" type="button">
            <div class="card-head">
              <div>
                <strong>${escapeHtml(client.name)}</strong>
                <p>${escapeHtml(client.type)} - ${money(Number(client.budget || 0))} - ${escapeHtml(client.zone || "Sin zona")}</p>
              </div>
              <span class="badge stage-${toneClass(client)}">${escapeHtml(client.stage)}</span>
            </div>
            <div class="chip-row">
              ${stale ? `<span class="mini-chip alert-chip">Pendiente seguimiento</span>` : ""}
              <span class="mini-chip priority-${escapeHtml(client.priority)}">${escapeHtml(client.priority)}</span>
              <span class="mini-chip">${escapeHtml(client.advisor || "Sin asesor")}</span>
              <span class="mini-chip">${daysSince(client.lastContact)} dias sin seguimiento</span>
            </div>
            <div class="card-meta">
              <p>${escapeHtml(client.nextAction || "Sin proxima accion")}</p>
              <strong>${formatDate(client.followUpDate)}</strong>
            </div>
          </button>
          <div class="quick-row">
            <select class="quick-stage" data-id="${client.id}" aria-label="Cambiar etapa de ${escapeHtml(client.name)}">
              ${stages.map((stage) => `<option ${stage === client.stage ? "selected" : ""}>${stage}</option>`).join("")}
            </select>
            ${
              appMode === "admin"
                ? `<select class="quick-advisor" data-id="${client.id}" aria-label="Cambiar asesor de ${escapeHtml(client.name)}">
                    ${advisors.map((advisor) => `<option ${advisor === client.advisor ? "selected" : ""}>${escapeHtml(advisor)}</option>`).join("")}
                  </select>`
                : ""
            }
            ${client.phone ? `<a class="quick-whatsapp" href="https://wa.me/${phoneForWhatsapp(client.phone)}" target="_blank" rel="noreferrer">WhatsApp</a>` : ""}
          </div>
        </article>`;
    })
    .join("");

  elements.list.querySelectorAll(".card-select").forEach((card) => {
    card.addEventListener("click", () => {
      selectedId = card.dataset.id;
      render();
    });
  });

  elements.list.querySelectorAll(".quick-stage").forEach((select) => {
    select.addEventListener("change", () => updateStage(select.dataset.id, select.value));
  });
  elements.list.querySelectorAll(".quick-advisor").forEach((select) => {
    select.addEventListener("change", () => updateAdvisor(select.dataset.id, select.value));
  });
}

function advisorOptions(includeInactive = false) {
  const directoryNames = advisorDirectory.filter((advisor) => includeInactive || advisor.active !== false).map((advisor) => advisor.name);
  const clientNames = clients
    .map((client) => client.advisor)
    .filter(Boolean)
    .filter((name) => includeInactive || advisorDirectory.find((advisor) => advisor.name === name)?.active !== false);
  const advisors = [...new Set([...directoryNames, ...clientNames])].sort();
  if (!advisors.includes(myAdvisorName)) advisors.unshift(myAdvisorName);
  return advisors;
}

function renderDetail() {
  const client = clients.find((item) => item.id === selectedId);
  if (!client) {
    elements.detail.innerHTML = `<p class="empty-state">Selecciona un lead para ver sus datos, etapa y siguiente accion.</p>`;
    return;
  }

  const stale = isStale(client);
  elements.detail.innerHTML = `
    <span class="badge stage-${toneClass(client)}">${escapeHtml(client.stage)}</span>
    ${stale ? `<span class="detail-alert">Pendiente seguimiento</span>` : ""}
    <h3>${escapeHtml(client.name)}</h3>
    <p>${escapeHtml(client.type)} - ${money(Number(client.budget || 0))}</p>
    <div class="detail-grid">
      <div class="detail-row"><strong>Telefono</strong><span>${escapeHtml(client.phone || "-")}</span></div>
      <div class="detail-row"><strong>Zona</strong><span>${escapeHtml(client.zone || "-")}</span></div>
      <div class="detail-row"><strong>Propiedad</strong><span>${escapeHtml(client.propertySent || "-")}</span></div>
      <div class="detail-row"><strong>Fuente</strong><span>${escapeHtml(client.source || "-")}</span></div>
      <div class="detail-row"><strong>Prioridad</strong><span>${escapeHtml(client.priority || "-")}</span></div>
      <div class="detail-row"><strong>Creacion</strong><span>${formatDate(client.createdAt)}</span></div>
      <div class="detail-row"><strong>Ultimo contacto</strong><span>${formatDate(client.lastContact)}</span></div>
      <div class="detail-row"><strong>Dias sin seguimiento</strong><span>${daysSince(client.lastContact)}</span></div>
      <div class="detail-row"><strong>Proximo seguimiento</strong><span>${formatDate(client.followUpDate)}</span></div>
      <div class="detail-row"><strong>Asesor</strong><span>${escapeHtml(client.advisor || "-")}</span></div>
      <div class="detail-row"><strong>Estado cliente</strong><span>${escapeHtml(client.clientStatus || "-")}</span></div>
    </div>
    <hr />
    <strong>Recordatorio</strong>
    <p>${escapeHtml(client.nextAction || "Sin accion registrada")}</p>
    <strong>Comentarios</strong>
    <p>${escapeHtml(client.comments || "Sin comentarios")}</p>
    <div class="detail-actions">
      <button class="secondary-button" id="editSelected">Editar</button>
      ${client.phone ? `<a class="primary-button link-button" href="https://wa.me/${phoneForWhatsapp(client.phone)}" target="_blank" rel="noreferrer">WhatsApp</a>` : ""}
    </div>
  `;

  document.querySelector("#editSelected").addEventListener("click", () => openForm(client));
}

function openForm(client = null) {
  elements.form.reset();
  elements.deleteButton.hidden = !client;
  elements.dialogTitle.textContent = client ? "Editar lead" : "Nuevo lead";

  const fields = [
    "clientId",
    "createdAt",
    "name",
    "phone",
    "type",
    "budget",
    "zone",
    "propertySent",
    "source",
    "priority",
    "lastContact",
    "nextAction",
    "followUpDate",
    "stage",
    "advisor",
    "clientStatus",
    "comments"
  ];
  fields.forEach((field) => {
    const input = document.querySelector(`#${field}`);
    input.value = client?.[field === "clientId" ? "id" : field] ?? "";
  });

  if (!client) {
    document.querySelector("#createdAt").value = today();
    document.querySelector("#type").value = "Venta";
    document.querySelector("#source").value = "WhatsApp";
    document.querySelector("#priority").value = "Media";
    document.querySelector("#nextAction").value = "Dar seguimiento";
    document.querySelector("#stage").value = "Nuevo Lead";
    document.querySelector("#advisor").value = myAdvisorName;
    document.querySelector("#clientStatus").value = "Activo";
    document.querySelector("#lastContact").value = today();
    document.querySelector("#followUpDate").value = addDays(1);
  }
  const advisorInput = document.querySelector("#advisor");
  if (appMode === "advisor") {
    advisorInput.value = currentAdvisor;
    advisorInput.disabled = true;
  } else {
    advisorInput.disabled = false;
  }
  elements.dialog.showModal();
}

function closeForm() {
  elements.dialog.close();
}

function saveClient(event) {
  event.preventDefault();
  const id = document.querySelector("#clientId").value || crypto.randomUUID();
  const previous = clients.find((client) => client.id === id);
  const formData = {
    id,
    createdAt: document.querySelector("#createdAt").value || today(),
    name: document.querySelector("#name").value.trim(),
    phone: document.querySelector("#phone").value.trim(),
    type: document.querySelector("#type").value,
    budget: Number(document.querySelector("#budget").value || 0),
    zone: document.querySelector("#zone").value.trim(),
    propertySent: document.querySelector("#propertySent").value.trim(),
    source: document.querySelector("#source").value,
    priority: document.querySelector("#priority").value,
    lastContact: document.querySelector("#lastContact").value,
    nextAction: document.querySelector("#nextAction").value,
    followUpDate: document.querySelector("#followUpDate").value,
    stage: document.querySelector("#stage").value,
    advisor: appMode === "advisor" ? currentAdvisor : document.querySelector("#advisor").value.trim() || myAdvisorName,
    clientStatus: document.querySelector("#clientStatus").value,
    comments: document.querySelector("#comments").value.trim(),
    updatedAt: Date.now()
  };

  const existingIndex = clients.findIndex((client) => client.id === formData.id);
  if (existingIndex >= 0) {
    clients[existingIndex] = formData;
    recordActivity(formData.advisor, activityFromChange(previous, formData), formData.name);
  } else {
    clients.unshift(formData);
    recordActivity(formData.advisor, "registro nuevo lead", formData.name);
  }
  selectedId = formData.id;
  persist();
  closeForm();
  render();
}

function activityFromChange(previous, current) {
  if (!previous) return "registro nuevo lead";
  if (previous.stage !== current.stage) {
    if (current.stage === "Visita agendada") return "agendo visita";
    if (current.stage === "Negociacion") return "paso a negociacion";
    if (current.stage === "Cierre") return "cerro operacion";
    return `cambio etapa a ${current.stage}`;
  }
  if (previous.followUpDate !== current.followUpDate || previous.nextAction !== current.nextAction) {
    return "registro seguimiento";
  }
  return "actualizo cliente";
}

function updateStage(id, stage) {
  const client = clients.find((item) => item.id === id);
  if (!client) return;
  const previous = { ...client };
  client.stage = stage;
  client.clientStatus = clientStatusFromStage(stage);
  client.lastContact = today();
  client.updatedAt = Date.now();
  recordActivity(client.advisor, activityFromChange(previous, client), client.name);
  selectedId = id;
  persist();
  render();
}

function updateAdvisor(id, advisor) {
  const client = clients.find((item) => item.id === id);
  if (!client) return;
  client.advisor = advisor;
  client.updatedAt = Date.now();
  recordActivity(advisor, "recibio cliente asignado", client.name);
  selectedId = id;
  persist();
  render();
}

function deleteClient() {
  const id = document.querySelector("#clientId").value;
  const client = clients.find((item) => item.id === id);
  clients = clients.filter((item) => item.id !== id);
  if (client) recordActivity(client.advisor, "elimino cliente", client.name);
  selectedId = clients[0]?.id ?? null;
  persist();
  closeForm();
  render();
}

function money(value) {
  return new Intl.NumberFormat("es-MX", {
    style: "currency",
    currency: "MXN",
    maximumFractionDigits: 0
  }).format(value);
}

function formatDate(value) {
  if (!value) return "Sin fecha";
  return new Intl.DateTimeFormat("es-MX", {
    day: "2-digit",
    month: "short"
  }).format(new Date(`${value}T12:00:00`));
}

function toneClass(client) {
  if (client.stage === "Cierre" || ["Negociacion", "Apartado"].includes(client.stage)) return "green";
  if (client.stage === "Nuevo Lead") return "blue";
  if (client.stage === "Perdido" || client.clientStatus === "Frio" || isStale(client)) return "red";
  return "yellow";
}

function phoneForWhatsapp(phone) {
  const digits = phone.replace(/\D/g, "");
  return digits.length === 10 ? `52${digits}` : digits;
}

function escapeHtml(value) {
  return String(value)
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#039;");
}
